# Card 22 — batten-retry-rows

**Edit:**
- `internal/battenshed#InnerRunDeps`
- `internal/battenshed#ChildPublishFailure`
- `internal/battenshed#NewInnerRun`
- `internal/battenshed#recordMatchesHalt` -> `func recordMatchesHalt(record ChildPublishFailure, head string, status shedengine.Status, rows []string) bool`
- `internal/battenshed#innerRunProducer.publishRetryDecision`
- `internal/battenshed#retryPublishRow`
- `internal/battenshed#newRetryHarness`
- `internal/battenshed#retryHarness.blockedAtPublish`
- `internal/battenshed#TestInnerRun_PublishRetrySendsTheChildToItsTargetAndResumesItOnce`
- `internal/battenshed#TestInnerRun_PublishRetryDeclinesRecordsThatDoNotMatchTheHalt`
- `internal/battenshed/doc.go`
- `internal/battencli#readChildPublishFailure`
- `internal/battencli#TestReadChildPublishFailure_MapsTheRecord`
- `internal/battencli#readAgentActivity`
- `internal/battencli#TestReadAgentActivity_ReadsLiveRunsAndTheWaitMarker`
- `internal/battencli#battenCLI.wire`

**Uses:**
- `plan:internal/parentlock#ReadWait`
- `plan:internal/parentlock#WaitRecordPath`

**Intent:**
Point batten's automatic verify retry at the row that ran the failing verify (D1), and count a parent lock wait as a live wait (D12).

`InnerRunDeps.PublishRow` becomes `VerifyRows []string`, the child recipe's rows that run a verify and write the failure record.
`ChildPublishFailure` gains `Row`.
`publishRetryDecision` treats a child blocked at any row of `VerifyRows` as the retry's halt.
`recordMatchesHalt` additionally requires the halting entry's producer to equal the record's `Row`, so a plan verify failure at Merge-In and a `publish_verify` failure at Publish each retry from their own halt, and a record that names another row, or no row, leaves the halt to the operator.
An empty `VerifyRows` switches the retry off, as an empty `PublishRow` did.
battencli maps the record's `Row` in `readChildPublishFailure`, and `wire` tells `VerifyRows` as `loomshed.NameMergeIn` and `loomshed.NamePublish`.
The retry harness halts at Merge-In for a plan verify record, and the decline table gains a record whose row differs from the halt.

`readAgentActivity` reports `waitLive` when the task pair's parent lock wait record is live, beside the verify marker and the shuttle wait markers, so no quiet notice fires while the run waits its turn.
A wait record whose writer is dead is ignored.
Its test gains both rows.
`doc.go` names Merge-In beside Publish in the retry paragraph and the lock wait in the quiet-notice paragraph.

**ImpactSummary:** batten's retry follows the plan verify to Merge-In and its quiet notice respects a lock wait; `recordMatchesHalt` is re-signed with its one caller.

**Commit:** 22: batten: the verify retry follows the failing row, and a lock wait is a live wait
