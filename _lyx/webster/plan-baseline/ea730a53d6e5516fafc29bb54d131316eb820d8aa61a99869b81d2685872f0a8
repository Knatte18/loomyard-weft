# Card 4 — batten-retry-merge-in-target

**Edit:**
- `internal/battenshed#retryHarness`
- `internal/battenshed#newRetryHarness`
- `internal/battenshed#retryHarness.pausedAtTarget`
- `internal/battenshed#retryHarness.wantGotos`
- `internal/battenshed#retryHarness.writeMarker`
- `internal/battenshed#retryTarget`
- `internal/battenshed#TestInnerRun_PublishRetrySendsTheChildToItsTargetAndResumesItOnce`
- `internal/battenshed/doc.go`

**Intent:** D2's batten side: batten's one automatic retry reads its target from the failure record, so a not-owned darn failure, whose record names `Merge-In` (card 3), sends the child to Merge-In and resumes it, and a second failure halts with the retry spent and the not-owned reason in the blocked notice.
No production code changes: `publishRetryDecision` already takes `record.Target`, `lyx loom goto --to Merge-In` is admitted from a halt at Merge-In or Publish (`gotoAdmitted` admits the reference row and every row before it on a blocked run), and the blocked notice already carries the child's error.

`retryHarness` gains a `target` field, which `newRetryHarness` sets to `retryTarget` and writes into the record; `pausedAtTarget`, `wantGotos` and `writeMarker` read `h.target` instead of the constant, so a test can retarget the harness.
`retryTarget`'s comment says it is the default target.

`TestInnerRun_PublishRetrySendsTheChildToItsTargetAndResumesItOnce` becomes table-driven over the halting row and the record's target, each row running the existing sequence of goto, resume and second halt:
- a halt at Publish whose record targets `Webster`, today's case;
- a halt at Merge-In whose record has row Merge-In and targets `Merge-In`;
- a halt at Publish whose record targets `Merge-In`.
In each row the goto names the row's target, the resume runs once, and the second halt at the same row, with the child's status error set to a not-owned reason such as `landing gate failed outside the task's impacted set, in github.com/Knatte18/loomyard/internal/x, after merging parent branch "main" (exit code 1)`, sends one notice holding `blocked at <row>`, `automatic Publish verify retry already ran` and `outside the task's impacted set`, with no second goto.
The reason leads with the not-owned words because the notice keeps only the error's first `noticeErrorMax` characters, which card 2's lead satisfies.

`doc.go`'s retry paragraph says the target is the record's: a not-owned darn failure names Merge-In, so the retry re-enters Merge-In, which merges a moved parent and re-runs the failed gate, and a second failure at Merge-In or Publish gets the usual blocked notice with the retry spent and the record's reason, which says the failure lies outside the task's impacted set.

The card adds no retry and no route: the one retry stays the run's only one, now pointed at Merge-In for a not-owned failure.

**ImpactSummary:** test-harness and doc change inside `internal/battenshed`; the other retry tests keep the default target through `newRetryHarness`.
