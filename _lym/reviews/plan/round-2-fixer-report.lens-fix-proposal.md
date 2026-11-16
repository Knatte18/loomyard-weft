```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each review proposal the fixer carried out (fix-proposal lens).

Proposals carried out as proposed and challenged: F1, F2, F3, F6 and F7. F4 and F5 are partly fixed differently; I checked only the parts taken from the proposal.

**MEDIUM — card 13, `checkStoppedBatch`: "runs `checkTreeAndHead` (card 10) with the `head_sha` of the stopped batch's last parsed report"**
- The head check written into the re-entry path (from F3's proposal) rejects the very case D7's no-spawn path exists for: a parent or arbiter repair after the stop.
- Concrete case: the last-rung attempt fails the clean-tree check. The arbiter or parent commits the leftover paths and re-steps. HEAD is now a new commit on top of the report's `head_sha`, and it is not a parent merge.
- `checkTreeAndHead` then fails with "head_sha not HEAD", so the batch never records done with no spawn and always spends a last-rung attempt.
- This also contradicts card 13's kept `TestRun_Reentry` row "records done with no spawn, also after the arbiter edits a pending card", whenever that plan edit is committed on the task branch.
- A stopped batch whose last attempt left no parsed report (an `OutcomeTimeout`, an unparseable report) also always fails. The new sentence states this outright: "a batch whose last attempt left no parsed report fails it".
- Evidence: card 10's head rule accepts only "equal to HEAD, or HEAD a merge commit whose first parent it is and that `Git.MergeRejection` accepts". Card 24 makes the arbiter the actor who repairs a stopped row.
- The re-entry needs the clean-tree check plus `PostBatchChecks` against the recorded start commit, with no report-head equality.

**MEDIUM — card 13, `checkStoppedBatch` returns `([]string, error)` and `recordDoneWithoutSpawn` "marks the batch record terminal done with no digest"**
- A batch done through the no-spawn path loses the deviations and departures that a batch done through `endAttempt` records.
- `runplan.PostBatchChecks` returns `(deviations, warnings []string, err error)` (`internal/runplan/postbatch.go:241`). Card 10's pass records "its deviations, departures (`runplan.BatchDepartures`) and card trail".
- The proposed signature of `checkStoppedBatch` returns only failing reasons, and `recordDoneWithoutSpawn` takes no deviations. So a pass whose post-batch checks re-baselined the plan or recorded a departure writes a done record without them.
- The two done paths now write different record shapes for the same verdict, which is the drift F3 meant to prevent.

**LOW — card 12, the new `TestRun` row "so a second acquire of the state-mutation lease would hang the test"**
- A regression of F1's deadlock shows up as a hang until the `go test` timeout, not as a failing assertion naming the cause.
- The row as proposed pins the bug only through the gate's timeout.
- A bounded probe would fail fast and name the cause: after the begin returns, `lock.AcquireWriteLockWithin` on `mutate.lock` succeeds (`internal/lock/lock.go:62`). Alternatively, run the begin under a test deadline.

No breakage found for F2's clear in `endAttempt`'s done branch, F6's explicit `WeaverRun` wiring, or F7's row deletion. Card 5's `AttemptState.Number` keeps the attempt-number contract, and card 12 and card 13's `TestRun_Reentry` rows still hold it.

Tool calls:
1. Grepped card 4 for `RecordBeginPartition` and `PendingPartition` and lease use. My context lacked card 4, and I needed to know whether the begin's other calls also take the lease.
2. Read card 24 and grepped the overview for D7. My context lacked who repairs a stopped row and whether that repair moves HEAD.
