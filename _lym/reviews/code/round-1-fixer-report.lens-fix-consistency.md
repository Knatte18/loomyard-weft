Findings: no findings.

- **F1 (`cli_test.go`):** The slice and the comment both now name the parent-review group, and the group's `Find` name matches `parentReviewGroup`.
- **F8 (`parentreview.go`):** The `AnswerParentAsk` doc comment still holds. Every refusal is a `*ParentAnswerRefusal`, none writes a report, and the past-deadline refusal is now returned only when its timeout state was written.
- **F2–F4, F6 (`loomcli/parentreview_test.go`):**
  - Every remaining use of `wantFindings` was replaced by `len(tt.wantReport)`.
  - `wantErr` became a slice, and both the no-ask row and the new cut-short row use the new form.
  - The new rows' refusal strings match the strings in `parentreview.go` (`not YAML holding a findings list`, `read the --findings-file`, `could not be resolved`, `the wait was cut short`).
  - Minor and not a finding: the `TestParentReviewAwaitVerb` doc comment doesn't mention the cut-short refusal.
- **F5, F7 (`burlerengine/parentreview_test.go`):** The `//testtiming:keep` line matches the existing format used in `contracts/stencils/parentdirective_test.go`. The unknown-state rows agree with the `Ended()` doc comment.

Calls made:
1. `grep -rn "testtiming:keep" --include=*.go .`: I wanted the repo's existing marker format to compare against the line the fixer added, and my context held no other marker.
