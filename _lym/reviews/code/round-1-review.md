---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: handler
    location: internal/landingshed/prreview.go:323
    summary: With the setting on, the run does not wait at PR-Review for the orch's answer; the operator's ending or an approval can finish the run before the orch has answered
    fix: In internal/landingshed/prreview.go record the orch's answer, for example a ParentReviewAnswered flag in pullrequest.Record written when the orch's satisfied answer arrives, and make judge hold (return not ended) while the record has ParentReviewAsked set and no satisfied answer, so a merge, approval or pr-continue does not end the row until the orch has answered satisfied or its findings are fixed; then correct the doc.go line "The row's endings do not wait on the orch's answer". If the author means the operator's ending to override the orch, record that with lym loom decision add and reword the doc.go and overview.md claims to say the row asks and does not gate.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/prreview.go:359
    summary: A successful post followed by a failed write of ParentReviewAsked returns an error, so the driver re-steps and the orch gets the same request twice
    fix: In askParentReview log the failed updateRecord with logger.Warn and return nil, as the other failure paths of that function do, and add a row in prreview_session_test.go for a record write that fails after a successful post asserting a single ask; or keep the error and add that row asserting the row's error.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/landingdeps.go:419
    summary: The ParentReviewWanted and AskParent closures have no test; the wiring seam tests only list them as filled by run.go, so a task's run_settings reaching the setting is not pinned
    fix: Add one case to the existing loomcli landing-deps test that builds pullRequestDepsGuardedBy over a fixture hub and asserts ParentReviewWanted reads true when the task's run_settings layer sets pr_review_ask_parent and false otherwise, without a new test file.
---

### [MEDIUM] The run does not wait on the orch's answer

**Location:** internal/landingshed/prreview.go:323, internal/landingshed/doc.go (new line "The row's endings do not wait on the orch's answer")
**Issue:** The board entry says that with the setting on the run waits at PR-Review until the orch answers satisfied or sends findings.
The diff mails the request once and then enters the unchanged `wait`, whose `judge` ends on a merge, an approval or the operator's `pr-continue`.
Nothing records the orch's answer, and `doc.go` states the row's endings do not wait on it, so the diff makes the requested behavior only in the sense that the row already waited for the operator.

### [LOW] A failed record write after a successful post sends the request twice

**Location:** internal/landingshed/prreview.go:359
**Issue:** `askParentReview` returns the `updateRecord` error after `AskParent` posted, the driver re-steps, and the flag is still unset, so the orch receives the request again.

### [LOW] The loom wiring of the setting and the ask is untested

**Location:** internal/loomcli/landingdeps.go:419
**Issue:** The shed tests use fakes for both seams, and the seam tests list them as filled by run.go without asserting what fills them, so a closure that read the wrong layer or the wrong notifier would pass every test.

## Rejected

- The code-tests lens's note that the nil guard `ParentReviewWanted == nil || AskParent == nil` has no row: the guard is a tolerated default the fixtures never reach, and pinning it adds a test for no behavior the entry asks for.
- The code-tests lens's note that the claim "a review started afresh asks again" and the no-ask closed, merged and `ReviewEnded` paths have no row: a fresh record is written without the flag by construction, and those paths never reach the ask, so a row would repeat the open-path rows.
- The code-tests lens's note that the default off has no dedicated row: the template line and the Go zero value both read false, and `ParentReviewOffAsksNone` pins the off behavior.
- The code-tests lens's F3 (no row for a failed record write after a post) is folded into F2's fix.
- The size of the rewrites in `internal/battencli/wire_seams_test.go` and `internal/loomcli/wiring_seams_test.go`: they are alignment changes that gofmt forces when the longest map key grows, not behavior.
- `internal/pullrequest/record.go` and `internal/testkit/envkit/envkit.go`: the flag is what keeps the ask to once per record, and the fixture fills the new seams, so both follow from the entry.
- The `docs/overview.md` and `landingshed/doc.go` claims match the code, apart from the line F1 names.
- No `Pattern-Departure` trailer is on either commit, and the diff touches no `PATTERN.md` or `pattern/` file.
