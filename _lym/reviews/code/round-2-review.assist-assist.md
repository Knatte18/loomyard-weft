---
verdict: APPROVED
findings:
  - id: F5
    severity: LOW
    class: design
    location: internal/landingshed/prreview_test.go:599
    summary: The optional-seam test takes its cases from the production exemption list, so removing ParentReviewWanted or AskParent also removes its test case and leaves the original regression risk uncovered.
    fix: In TestPRReview_Call/OptionalSeamsLeftUnsetRunTheRow, replace the loop over optionalPullRequestSeams with independent named unsetters for ParentReviewWanted, AskParent, BeamRecord and DropBeamRecord, following the required-seam unsetters beside it. Keep the assertion that Call returns no error for each unset field.
    origin: lens:code-tests
    carries: F5
  - id: F6
    severity: LOW
    class: consistency
    location: _lym/landing/summary.md:24
    summary: The summary reports 2574 production comment lines, but the final tree measures 2576; the prior measurement finding remains open.
    fix: In _lym/landing/summary.md change the production comment-line endpoint from 2,574 to 2,576, measured by go run ./tools/codestats -root internal/landingshed at HEAD. If further edits change the count, rerun that command after them and use its final production comment count.
    origin: handler
    carries: F6
prior:
  - id: F1
    status: resolved
  - id: F2
    status: resolved
  - id: F3
    status: resolved
  - id: F4
    status: resolved
  - id: F5
    status: open
  - id: F6
    status: open
---

### [LOW] Optional-seam cases disappear with the exemption

**Location:** internal/landingshed/prreview_test.go:599

**Issue:** The test and `checkWiring` use the same list to decide which fields are optional.
Removing `ParentReviewWanted` or `AskParent` from that list removes its test case too.
The fixture supplies both fields in every remaining case, so the regression still escapes coverage.
The new cases exercise today's list but do not independently pin the optional-field contract.

### [LOW] Comment count still predates the final tree

**Location:** _lym/landing/summary.md:24

**Issue:** A fresh `go run ./tools/codestats -root internal/landingshed` reports 2,576 production comment lines.
The summary says 2,574.
The measured 5,146 production code lines and 11,014 test code lines match the summary.
