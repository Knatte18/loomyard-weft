---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/burlerengine/parentreview_test.go:138
    summary: the F8 ordering in AnswerParentAsk, a failed record write returned ahead of the past-deadline refusal, has no test
    fix: "In TestAnswerParentAsk, or a sibling row set in internal/burlerengine/parentreview_test.go, record an open ask whose deadline is parentAskBase, then make the record's directory unwritable (chmod files.Dir to 0o500 via t.Chmod or os.Chmod with a t.Cleanup restore, skipping on Windows and when running as root). Call AnswerParentAsk at parentAskBase.Add(time.Minute) and assert the error is not a *ParentAnswerRefusal, that errors.As against the refusal fails, and that the record still reads the ask open. Without a feasible way to fail the write, state that in a comment and leave the row out."
---

### [LOW] The write-failure ordering fix has no test

**Location:** internal/burlerengine/parentreview_test.go:138

**Issue:**
Focus question 1 is settled in the production code.
In `AnswerParentAsk` (`internal/burlerengine/parentreview.go:207-244`), the past-deadline branch sets `ask.State = ParentAskTimeout`, stores the refusal and returns `record, nil`.
`state.UpdateJSON` (`internal/state/state.go:173-178`) then writes the record, and a failed write comes back as a non-nil error.
That error is not `errParentAskUnchanged`, so the check at line 237 returns it wrapped, ahead of the `refusal != nil` check at line 240.
The caller therefore never sees a refusal claiming the ask is now `timeout` when the write failed.
A failing report write on the answer path takes the same route.

No test exercises that ordering.
The commit that fixed it (`32bd1767`) changed production code only.
`TestAnswerParentAsk` has a row for the past-deadline refusal, but every row runs against a writable record, so reverting the fix would leave the whole suite green.
This is the test-economy case of a behavior added with no coverage.
It is LOW because the guard is one condition and the failure needs an unwritable record.

Focus question 2 is also settled: the rows added in the fix commits each pin behavior no other row covers.
- The loomcli rows for an empty file, a malformed file, a missing path and a whitespace-only `change` each reach a distinct branch of `readParentFindings`.
- The unresolved-parent and wrong-caller rows reach distinct branches of `parentAnswerCallerRefusal`.
- The no-ask-and-no-parent row pins the operator-shell wording of `parentReviewWayDecision`.
- The await rows for a cut-short wait and for a not-sent reason each pin an output the other rows do not.
- The burlerengine unknown-state rows pin `Ended()` through `AnswerParentAsk` and through `ExpireParentAsk`, which are separate code paths.
- Rows that assert the written report are the `findings file` row and the burlerengine rows.
  The `satisfied` row in loomcli asserts only the envelope and ask state, but the burlerengine "no findings make a satisfied report" row reads the report back, so no gap results.
- `TestAnswerParentAsk_RacingTheDeadline` carries a `//testtiming:keep` marker whose reason names what it pins.

Beyond the focus, I found no deviation from the decision record in D4's verb surface, caller guard, refusals or `P1…` numbering, or in D5's forward-only moves, once-a-second poll and `ready` answer shape.
