# Card 19 — burler-forkcount-retry

**Edit:**
- `internal/shuttleengine#Result`
- `internal/shuttleengine#Run.finalize`
- `internal/shuttleengine#TestWait_LogsSessionStateBesideItsClassification`
- `internal/burlerengine#ReviewResult`
- `internal/burlerengine#reviewResultOf`
- `internal/burlerengine#TestEngine_Review_Outcomes`
- `internal/shedadapters#BurlerReviewProducer.Call`
- `internal/shedadapters#BurlerReviewProducer.probeLiveReview`
- `internal/shedadapters#TestBurlerReviewProducer_Call_FailureExits`
- `internal/shedadapters#TestBurlerReviewProducer_Call_ProbeAndResume`
- `internal/shedadapters/doc.go`
- `internal/shuttleengine/waitshadow.go`
- `internal/shuttleengine/doc.go`

**Create:**
- `plan:internal/shedadapters#forkMismatchError` -> `func forkMismatchError(name string, round int, first, second burlerengine.ReviewResult, firstErr, secondErr error) error`

**Intent:** D9: a review round whose chair spawned a fork count other than the one requested is retried, and its failure says how the chair ended.

`shuttleengine.Result` gains `SessionStates []SessionState`, the session states the wait's session fold recorded for the run, in order, filled by `Run.finalize` from the fold's history when the wait read any; it is a report only, and shuttle's completion test is untouched (`PATTERN-completion-signal`).
`burlerengine.ReviewResult` gains `SessionStates`, copied from the chair's result by `reviewResultOf`, so it reaches burler on every ending, the audit error included, beside `Review.StrandGUID`.

In `BurlerReviewProducer.Call`'s attempt loop, a review that fails with `burlerengine.ErrClusterForksMissing` (fewer forks than requested, more, or no fork audit) on attempt 1 is retried like a died or timed-out attempt: the round's outputs, the chair's review file included, are archived by the next attempt's archive step and attempt 2 runs.
A mismatch on attempt 2 is a hard failure through the failure exit, with `forkMismatchError`'s text: both attempts' errors (each naming the requested and spawned counts), the chair's strand and its session states, for example an interrupt before `done`; the halt note carries it.
Any other fork violation, on either attempt, stays a hard failure at once.
`probeLiveReview` treats a resumed reviewer whose review fails with the sentinel as a died one: it falls through to a fresh attempt 1.
`doc.go`'s review-row paragraph names the retry.

Shadow mode still never changes a classification: `waitshadow.go`'s header and `shuttleengine`'s `doc.go` say the fold now also reports its states on `Result.SessionStates`, as a record the loop never branches on.
Tests: `TestWait_LogsSessionStateBesideItsClassification` asserts the done run's `Result.SessionStates` holds the logged states in order, and its shadowed-versus-plain equality check clears `SessionStates` on both results before comparing, as it already clears gate spans, so it still pins that the fold changes nothing else; `TestEngine_Review_Outcomes` adds a cluster row whose audit has too few forks, asserting the error wraps the sentinel and the result carries the strand and the states; `TestBurlerReviewProducer_Call_FailureExits` adds rows: a mismatch then a done review is retried and accepted; a mismatch on both attempts fails naming both counts, the strand and the states, with the round archived; a no-audit mismatch then done is retried; a mismatch then a died attempt 2 fails as two failed attempts naming the mismatch and the death; a different fork violation is not retried; `TestBurlerReviewProducer_Call_ProbeAndResume` adds a row where the resumed reviewer fails with the sentinel and a fresh attempt 1 runs.

**ImpactSummary:** Additive result fields in shuttle and burler; one new retry branch in the review row's attempt loop.

**Commit:** 19: burler review: retry a round whose chair spawned the wrong fork count, and name how the chair ended
