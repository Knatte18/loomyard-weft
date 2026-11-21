---
round: 1
focus:
  - "`mergeOnGitHub` in `internal/landingshed/finalize_pr.go`: does the `ReadMergeState` poll, which runs its own `MergeRetryWindow`, keep Finalize's wait bounded as the issue's \"bounded time\" and the doc's \"within the same minute\" claim? Check the worst case of poll plus the merge retry window."
  - "`mergeOnGitHub` retry path: when `MergeableNow` is true yet the merge keeps being refused as `RefusalOther`, does the loop end at the deadline with the resume-only halt, and does each retry re-read state without unbounded growth of waits or reads?"
  - "`mergeRefusalReason`: do the halt texts name the actual cause as the issue's Expected asks, in both the beam-record branch and the task branch, and does the resume-only text hold for a state that can merge?"
  - "`ReadMergeState` failure inside the merge loop: is a read error halted through `githubFailure` correctly, and does the `read, err :=` shadowing of the outer `err` hide anything?"
  - "Root-Cause trailer of commit 9f602d61: does it name the defect the diff removes, and does the diff touch it?"
  - "The `internal/pullrequest` rename of `beamHead` to `wantHead` and the cause text change to \"the expected head\": do other callers or tests of `mergeCauses` (beam pr status, mark-ready) still match the new text, and is no old name left?"
  - "Test coverage in `finalize_pr_test.go`: do the new cases cover unknown-then-mergeable, a definite refusal with named causes, and a poll that outlasts the window, with expected values from literals?"
  - "`doc.go` landingshed Finalize section: does each new sentence match the code's behavior, and does the diff land every doc the repository's constraints name for a Finalize behavior change?"
---

Seed focus from the decision record's board entry (issue #161) and the committed diff.
