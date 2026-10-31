---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/landingshed/prreview_test.go:435
    summary: PR-Review has no test row for an empty credit block, so the body-alone branch of bodyWithCredit is uncovered
    fix: In internal/landingshed/prreview_test.go add a TestPRReview_Call row whose Deps.Config has Credit false (copy prReviewTestDeps and clear Credit) and assert the scripted Create call, or the Edit call, carries the trimmed description body alone ("Change body") with no trailing blank line or credit lines.
    origin: lens:code-tests
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/landingshed/prreview.go:227
    summary: D4 makes an engines-run read failure in PR-Review an error, but no PR-Review test makes EnginesRun fail
    fix: In internal/landingshed/prreview_test.go add a TestPRReview_Call row that replaces deps.EnginesRun with a func returning an error, and assert the call returns that error wrapped, with no FindOpen, Create or Edit call recorded.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: consistency
    location: internal/summaryparser/summary.go:87
    summary: LandingMessage still names its parameter trailers and its doc says trailers verbatim, but it now receives the plain-line credit block
    fix: In internal/summaryparser/summary.go rename the parameter trailers to credit and reword the doc to say it adds a blank line and the credit block verbatim when credit is non-empty, with an empty credit yielding CommitMessage alone.
    origin: lens:code-affected-set
  - id: F4
    severity: LOW
    class: consistency
    location: internal/loomcli/prcontinue.go:3
    summary: The file header says Finalize lands under its own gate and trailer, but Finalize no longer appends a trailer
    fix: In internal/loomcli/prcontinue.go line 3 change "its own gate and trailer" to "its own gate and credit block".
    origin: lens:code-affected-set
---

### [MEDIUM] No test for the empty credit block in PR-Review

**Location:** internal/landingshed/prreview_test.go:435
**Issue:** `prReviewTestDeps` fixes `Credit: true` with a non-empty block, and every PR-Review and merge-message test builds on it.
D4 and the D2 acceptance criterion say an empty block leaves the body alone.
Nothing in the PR-Review tests would fail if the empty-credit branch of `bodyWithCredit` broke.

### [MEDIUM] No test for the engines-run read failure in PR-Review

**Location:** internal/landingshed/prreview.go:227
**Issue:** D4 says PR-Review's read failure is an error, as at Finalize.
The code returns it before any pull request call, and the code-correctness lens ran that path as correct.
Only Finalize has a failing-read row (`finalize_test.go:340`); `prreview_test.go:445` always returns a list.

### [LOW] LandingMessage doc and parameter still say trailers

**Location:** internal/summaryparser/summary.go:87
**Issue:** The argument is now the credit block, written as plain lines.
`doc.go:199` says it is never a git trailer, which this comment contradicts.
Finalize at `finalize.go:195` is the one caller outside tests.

### [LOW] prcontinue.go header still names a trailer

**Location:** internal/loomcli/prcontinue.go:3
**Issue:** The header says Finalize lands "under its own gate and trailer".
The parallel phrase in `finalize.go` already reads "gate and credit block".

## Rejected

- The code-tests lens noted that the credit key categories (cross-pair, open map) have no direct test: rejected, a wrong category fails the strict load and `TestLoadConfig_TemplateDefaults` loads the template.
- The code-tests lens noted that the finding detail text in `description.go` has no assertion: rejected, the lens itself found a message string is not behavior worth a row.
- The code-tests lens noted that `LandingMessage` with an empty block has no Finalize row with `Credit` off: rejected, the empty-block row of `TestConfig_CreditBlock` and the empty-string case of `LandingMessage` cover the two halves.
- The code-correctness lens noted that a heading-less block opens with a space and that an engine text with an inner newline gets no item prefix on its second line: rejected, D2 permits both and the second needs a hand-edited value.
