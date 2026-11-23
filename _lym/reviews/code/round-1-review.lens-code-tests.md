```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test row that would fail if it broke.

**Behaviour → pinning test**

1. A `no-parent` latest ask under the unreached tag is answered and becomes `done` with a report. Pinned by `TestAnswerParentAsk` "a no-parent ask under the unreached tag is answered" and by `TestParentReviewAnswerVerb` "the parent answers a no-parent PR-Review ask at the head it named".
2. An `undelivered` latest ask under the tag is answered past its deadline, so the deadline is skipped for an unreached ask. Pinned by `TestAnswerParentAsk` "an undelivered ask … answered past its deadline" and by `TestParentReviewAnswerVerb` "the parent answers an undelivered PR-Review ask with findings".
3. The tag must match. Pinned by `TestAnswerParentAsk` "a no-parent ask under another tag is refused".
4. A call that names no tag is refused. Pinned by `TestAnswerParentAsk` "… refused when the call names no unreached tag". This row does not pin the `unreachedTag != ""` guard; see finding 2.
5. A cleared ask is refused. Pinned by `TestAnswerParentAsk` "a cleared no-parent ask …" and by `TestParentReviewAnswerVerb` "a cleared no-parent PR-Review ask is refused", which also asserts the text "already no-parent" and the tag.
6. A timed-out ask under the tag stays refused. Pinned by `TestAnswerParentAsk` "a timed-out ask under the unreached tag is refused".
7. Only the latest ask is answerable: an older unreached ask is refused once a newer ask exists. Pinned by `TestAnswerParentAsk` "an older no-parent ask … once a newer ask exists".
8. The verb passes `landingshed.PRReviewAskLineTag`. Pinned by the two answering rows of `TestParentReviewAnswerVerb` (behaviours 1 and 2), which fail on the merge base.
9. A satisfied answer to an ended ask stands at the head it named, so `pr-continue` proceeds. No new row covers the whole path from the answer to the standing. It rests on two pieces, which I judge sufficient:
   - the new rows assert that the ask becomes `done` and keeps its `Digest`;
   - the existing done-at-head rows of `TestReadPRReviewStanding` cover the standing. `ReadPRReviewStanding` itself is unchanged.
10. The answer and the expiry stay atomic under the one lock. `TestAnswerParentAsk_RacingTheDeadline` covers the deadline path with an empty tag. The unreached path runs inside the same `UpdateJSON` closure and adds no new lock site, so a separate race row would pin nothing new.
11. The mechanical call-site renames in existing tests (driftengine, landingshed, loomcli, shedadapters) pass `""`, which keeps the old behaviour. No assertion is widened or removed, so nothing is loosened.

**Findings**

1. **LOW, consistency:** `TestBouncer_Call_RefusesARecoveredConvergedOverParentFindings` (`internal/shedadapters/bouncer_parent_test.go:133`) is listed as contained by `TestBouncer_JudgeCall_AsksTheParentBeforeJudging` and `TestBouncer_Reentry`.
   - It pins things neither covering test asserts:
     - the recovered-verdict path, where a CONVERGED verdict is already on disk at Call entry;
     - the reason text "holds findings, which rule out CONVERGED";
     - no judge spawn, and zero commits.
   - The covering test `TestBouncer_JudgeCall_AsksTheParentBeforeJudging` runs the judge live (evidence at `bouncer_parent_test.go:62-115`).
   - Fix: keep the test and add `//testtiming:keep pins the recovered-verdict refusal's reason, the verdict's archive, and that no judge or commit runs` directly above its `func Test…` line.
2. **NIT, design:** no test row pins the `unreachedTag != ""` guard at `internal/burlerengine/parentreview.go:371`.
   - The no-tag row (`parentreview_test.go` "a no-parent ask under the tag is refused when the call names no unreached tag") uses an ask tagged `[pr-review request]`. The tag-equality check already refuses it, so deleting the guard leaves every row passing.
   - A row would pin it: a `no-parent` ask with an empty `Tag`, answered with `unreachedTag` empty, expects a refusal of kind `ParentAnswerEnded`.
3. **NIT, consistency:** test failure messages still name the removed `AnswerParentAsk()`, for example `"AnswerParentAsk() error = %v; want nil"`. The edited call sites now call `AnswerParentAskOrUnreached`.
   - Sites: `internal/shedadapters/bouncer_parent_test.go:73,155`, `internal/shedadapters/parentseat_test.go:144`, `internal/driftengine/judgeparent_test.go:185,268`, `internal/driftengine/verify_test.go:2075`, `internal/landingshed/prreviewask_test.go` (indirectly), `internal/loomcli/commitrecords_test.go:350`, `internal/loomcli/parentreview_test.go:637`, and `internal/burlerengine/parentreview_test.go:317,325,328,361,365,424,567,731`.
   - These messages report a call to a function that no longer exists.

Tool calls:
- Read `internal/shedadapters/bouncer_parent_test.go` lines 30–200 and the head of `TestBouncer_Reentry`. The redundancy report flagged the bouncer test as contained, and its assertions and those of its covering tests were not in context.
