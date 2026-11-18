---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/landingshed/prreview_test.go#TestPRReview_Call
    summary: The OptionalSeamsLeftUnsetRunTheRow subtest takes its cases from the production list optionalPullRequestSeams, so dropping ParentReviewWanted or AskParent from that list also drops its test case, and the row would refuse every run that leaves that seam nil with no test failing.
    fix: In TestPRReview_Call, subtest OptionalSeamsLeftUnsetRunTheRow, replace the loop over optionalPullRequestSeams with a table of its own that names ParentReviewWanted, AskParent, BeamRecord and DropBeamRecord as literal field names, one row each, and run one t.Run per row that sets only that field of fx.deps.PullRequest to its zero value and asserts that Call returns no error. Leave optionalPullRequestSeams and checkWiring in prreview.go unchanged.
    origin: assist:assist
    carries: F5
  - id: F2
    severity: LOW
    class: consistency
    location: _lym/landing/summary.md:23
    summary: The summary gives production comment lines as 2,568 to 2,574, but go run ./tools/codestats -root internal/landingshed at HEAD measures 2,576; the round's later comment commits, 403a33ba and 1248d199, changed the count after the figure was written.
    fix: In _lym/landing/summary.md, in the sentence on production comment lines, change 2,574 to 2,576. If a later edit changes any file under internal/landingshed, run go run ./tools/codestats -root internal/landingshed again on the final tree and write its production comment count there instead.
    origin: assist:assist
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

### [LOW] The optional-seam test takes its cases from the list it should pin

**Location:** internal/landingshed/prreview_test.go#TestPRReview_Call

**Issue:** `checkWiring` exempts a `PullRequestDeps` field by its name in `optionalPullRequestSeams`, and the new subtest loops over that same list.
A rename of a listed field is caught: `FieldByName` then yields an invalid value and `SetZero` panics.
A name dropped from the list is not caught, because its test case leaves with it.
The fixture wires every optional seam in the remaining cases, so `Call` still succeeds and the regression that prior F5 named still escapes the test.
The cases need a source independent of the production list.

### [LOW] The comment-line figure still predates the final tree

**Location:** _lym/landing/summary.md:23

**Issue:** A fresh `go run ./tools/codestats -root internal/landingshed` at HEAD reports 5,146 production code lines, 2,576 production comment lines and 11,014 test code lines.
The code and test figures match the summary, and so do the per-commit code counts of the table: 5,312, 5,213 and 5,131, measured at `fd8c599`, `5064cc5` and `eb3feb7`.
The base measures 5,418 code and 2,568 comment lines, as stated.
Only the comment endpoint, 2,574, is stale.

## Prior findings

- F1: `internal/shuttleparity/seats.go` declares `soloSessionHolder` once, with a doc comment, and the three catalog entries use it.
- F2: `landUnderLock` reads `parentHoldsWork` before `ownerChecks` and again after it returns nil.
  `landCheck` and `syncParent` take the second answer, and either failed read halts with `ownerHoldsReadReason`, which opens with `OwnerHaltPrefix`.
  The summary no longer claims the read serves checks that move nothing.
- F3: `Finalize.Call` wraps the parse error as `parse change description`, and `PRReview.pullRequestText` wraps it as `read the change description`, each as at the base.
  `completeSummary` returns the engines and regression-check errors unwrapped, and both callers wrap them as before.
  The fixer's different shape keeps each caller's text, so the finding is resolved.
- F4: `ParentAlign.readAttempts` reads `RowEntry` and returns its wrapped error before the `moveScratchAttempts` loop.
  `Finalize.readAttempts` still moves first, then resolves the entry, 0 with no `RowEntry` wired, both matching the base.
- F5: carried as F1 above.
  The subtest pins the current names but not a dropped one.
- F6: carried as F2 above.

## Observations below this round's bar

- No new finding was raised below BLOCKING, as the incremental-round rule requires.
  The second parent-holds read turns a failed read at the sync, an error at the base, into an owner-check halt, and the summary's opening claim that only one log message changed does not name it.
  No test drives a failure of that second read alone.
  None of these is BLOCKING, so none is filed.

## Focus departures

- "internal/landingshed/finalize_pr.go, Deps.landingSummary, with its callers Finalize.Call and PRReview.pullRequestText": `Deps.landingSummary` no longer exists; the round-1 fix replaced it with `Deps.completeSummary`, so the question was settled against that function and the same two callers.
