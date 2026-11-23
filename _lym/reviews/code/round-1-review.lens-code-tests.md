```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, I looked for the test row that would fail if it broke.

**Per behaviour, the test that pins it**

1. A satisfied answer at head still stands after later asks at head that ended undelivered, no-parent or timeout (the loop's `continue` past asks that are not done).
   Pinned by `TestReadPRReviewStanding/AFailedLaterAskAtTheHeadLeavesASatisfiedAnswerStanding` (prreviewask_test.go:140).
2. A satisfied answer at head still stands after a later open ask at head.
   Pinned by `AnOpenLaterAskAtTheHeadLeavesASatisfiedAnswerStanding` (prreviewask_test.go:150).
   The pending-post state follows the same skip branch and has no row of its own; that is no gap.
3. A later answer with findings at head replaces the satisfied answer (the `break`).
   Pinned by `ALaterAnswerWithFindingsAtTheHeadReplacesASatisfiedAnswer` (prreviewask_test.go:158).
   With `break` turned into `continue`, the loop would reach the older satisfied answer and the row would fail.
4. After an answer with findings at head, a later open ask at head gives awaiting.
   Pinned by `AnOpenAskAfterAnAnswerWithFindingsAtTheHeadAwaits` (prreviewask_test.go:166).
5. A cleared satisfied answer is skipped.
   Pinned by `AClearedSatisfiedAnswerAtTheHeadAsksAgain` (prreviewask_test.go:174).
   Dropping `ask.Cleared` from the loop's filter would return satisfied and fail the row.
6. The loop's tag filter is pinned by `ACodeReviewAskIsNeverReadAsAPRReviewAsk` (prreviewask_test.go:138), a satisfied answer at h under the code-review tag.
7. The loop's digest filter is pinned by `ASatisfiedAskAtAnOlderHeadAsksAgain` (prreviewask_test.go:133).
8. The loop's `State != ParentAskDone` filter: dropping it sends an open ask's empty `Report` to `ReadParentReport`.
   That fails the open-ask rows, for example `AnOpenAskAtTheHeadAwaits` (prreviewask_test.go:129).
9. The fallthrough when no answered ask exists at head (no ask, another head, cleared, awaiting before or past the deadline) is pinned by the rows that already existed (prreviewask_test.go:128–137).
10. An indirect change in `parentReviewAskVerb`'s awaiting refusal (internal/loomcli/parentreview.go:326–336): no test pins it.
    When a satisfied answer at head is followed by an open ask at head, the standing is now satisfied, not awaiting.
    So the verb records and posts another ask while one is still open, where before it refused and named the await verb.
    `TestParentReviewAskVerb` has no row for a satisfied answer followed by an open ask (internal/loomcli/parentreview_test.go:379–460).
11. The stencil line and the doc.go addition are static prose and need no test.

**Findings**

- **LOW, design. Location:** internal/loomcli/parentreview.go:326.
  **Summary:** The diff changes how `parentReviewAskVerb` behaves when a satisfied answer at head is followed by an open ask at head: it now records a new ask instead of refusing with the await verb. No test row pins that behaviour either way.
  **Evidence:** internal/loomcli/parentreview.go:326–336 (the verb refuses only on `PRReviewAwaiting`); internal/landingshed/prreviewask.go:112–123 (satisfied is returned before the latest ask is checked); internal/loomcli/parentreview_test.go:379–460 (no row with a done satisfied ask followed by an open ask).
  **Fix:** Add a row to `TestParentReviewAskVerb` holding a satisfied ask and then an open ask at the head, with the expected result taken from the decided contract.

Tool calls:
- `grep` of internal/loomcli/parentreview_test.go for the ask verb's test and its awaiting rows: the manifest did not include that verb's tests.
- `sed` of internal/loomcli/parentreview_test.go lines 379–470 to read `TestParentReviewAskVerb`'s rows: the manifest did not include them.
