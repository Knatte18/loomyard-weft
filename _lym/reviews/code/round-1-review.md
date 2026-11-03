---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: handler
    location: internal/landingshed/deps.go:114
    summary: "Deps.DriftVerdict's doc says the drift guard keeps it filled, but the diff lists Landing.DriftVerdict in envkit nilLegal, so the nil-seam guard no longer catches an unwired seam and no test pins the production wiring."
    fix: "In internal/testkit/envkit/envkit.go remove the Landing.DriftVerdict entry from nilLegal and give the Landing deps built at envkit.go:172 a stub DriftVerdict returning an empty string and nil. Add Landing.DriftVerdict to intentionallyNil in internal/loomcli/wiring_seams_test.go with the same reason as Landing.EnginesRun, since run.go assembles Env.Landing. Add a direct check in internal/loomcli that landingDeps returns a non-nil DriftVerdict which renders a seeded scope record, and one that the DescribeTable closure in wiring.go carries driftengine.Verdict into the prompt. If the guard is not to cover it, instead reword the deps.go comment so it no longer claims a guard keeps the seam filled."
  - id: F2
    severity: MEDIUM
    class: decision
    origin: handler
    location: internal/landingshed/describe.go:137
    summary: "The entry asks Describe to add Drift's outcome to the summary and to drop or rewrite a summary line asking for a check Drift made; the diff appends the section in Go and never writes it to summary.md, and a stale hand-check line survives whenever Describe is skipped, such as with the Code-Review perch off."
    fix: "Leave the code in place and defer this finding to the run's parent. Its only sound change is recording the departure from the amended entry, or refusing it, with lym loom decision add, which lies outside the fixer's write surface."
  - id: F3
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/landingshed/finalize.go:228
    summary: "Finalize mutates summary.Body so the section also reaches the GitHub merge title and message through bodyWithCredit, which the doc and change description claim, but no test row asserts it; the new Finalize row checks only MergeOptions.Message."
    fix: "Add a row to the Finalize pull-request merge test in internal/landingshed/finalize_test.go with Deps.DriftVerdict set, asserting the fake API's merge message carries the body, then the section, then the credit block."
  - id: F4
    severity: LOW
    class: decision
    origin: handler
    location: internal/landingshed/finalize.go:205
    summary: "The entry licenses the verdict in the summary, the pull request and the PR-Review prompt only; the diff also appends it to the landing commit message, which the entry does not ask for."
    fix: "Leave the code in place and defer this finding to the run's parent, to record or refuse the extra landing-commit change with lym loom decision add; the fixer does not revert it."
  - id: F5
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/landingshed/prreview.go:240
    summary: "Three added behaviours have no test row: the re-run refresh of an open pull request carrying the section once, withSection on a whitespace-only body, and a DriftVerdict error from sessionSpec."
    fix: "In internal/landingshed add a refresh row to TestPRReview_Call with DriftVerdict set that asserts the title and body update carries the section once; add a withSection row with an empty-body description and a verdict that asserts the body is the section alone; add a TestPRReview_Session row whose DriftVerdict seam errors and that asserts no session start."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/driftengine/verify_test.go:247
    summary: "requireCompareRecorded pins BaselineHead and ComparedHead to the same value aaa, so swapping the two arguments in recordCompare would still pass."
    fix: "In internal/driftengine/verify_test.go give the fixture a capture head that differs from its HEAD in the passing-compare and agent-Done rows, and have requireCompareRecorded take the two expected commits separately."
---

### [MEDIUM] Deps.DriftVerdict is exempt from the guard its doc cites

**Location:** internal/landingshed/deps.go:114, internal/testkit/envkit/envkit.go:88
**Issue:** The doc comment on `Deps.DriftVerdict` says `landingDeps` fills it and its drift guard keeps it filled.
Commit `71d8c0a1` lists `Landing.DriftVerdict` in `nilLegal`, which `NilSeams` skips, so the guard never fails on a nil `DriftVerdict`.
A nil seam silently drops the regression check from the pull request, the failure this task exists to remove.
No test checks that `landingDeps` fills the seam, or that the Describe table closure in `wiring.go:816` reads the verdict.
The lens-code-tests report holds the same gap as its F6.

### [MEDIUM] The summary and stale-line parts of the entry are met only when Describe runs

**Location:** internal/landingshed/describe.go:137, internal/landingshed/finalize_pr.go:93, contracts/stencils/landing/landing-template-describe.md:23
**Issue:** The entry's Expected says Describe adds the outcome to the summary, and that a summary line asking for a check Drift made is dropped or rewritten.
The diff keeps the section out of `summary.md` and appends it in Go, and the change description records this as a design decision.
The pull request does carry the outcome, which meets the first bullet's end.
The third bullet holds only through the Describe stencil's instruction: with Describe skipped, as with the Code-Review perch off, the writer's stale "run the compare by hand" line stays above the appended section.
The change description admits that case.
This is a recorded departure from the amended entry that no `Added after Discussion` entry covers, so the parent decides it.
The lens-code-affected-set report holds the same gap as its F-A.

### [MEDIUM] The GitHub merge message carrying the section is untested

**Location:** internal/landingshed/finalize.go:205, internal/landingshed/finalize.go:228
**Issue:** `summary.Body` is rewritten once, so the same body feeds `LandingMessage` and the `bodyWithCredit` call that `landUnderLock` hands to the GitHub merge.
`TestFinalize_MergeOptionsCarriesComposedMessage` asserts only `MergeOptions.Message`.
The doc and the change description say the GitHub merge message carries the section, and no row would fail if it did not.

### [LOW] The landing commit message goes beyond the entry

**Location:** internal/landingshed/finalize.go:205
**Issue:** The entry names the summary, the pull request and the PR-Review prompt.
The diff also adds the section to the landing commit message, and says so in `doc.go` and the final-summary spec.
The spec already ties the commit message to the pull request body, which supports the choice.
The entry does not license it, so the parent records or refuses it.

### [LOW] Added paths with no test row

**Location:** internal/landingshed/prreview.go:240, internal/landingshed/finalize_pr.go:93, internal/landingshed/prsession.go:125
**Issue:** The refresh of an open pull request computes its body through `openPullRequest` as the first publish does, and only the first publish has a row with a verdict.
`withSection`'s whitespace-only body branch and the error return from `driftVerdict()` in `sessionSpec` have no row either.

### [LOW] The compare-recorded assertion cannot tell the two commits apart

**Location:** internal/driftengine/verify_test.go:247
**Issue:** The fixture's capture head and HEAD are both `aaa`, so `requireCompareRecorded` passes whichever commit `recordCompare` stores in `BaselineHead` and `ComparedHead`.
A swap at `verify.go:285`, or `told.BaselineHead` standing in for HEAD, would not fail any row.

## Rejected

- Partial scope with an empty skip list renders `partial, skipping .` (lens-code-correctness F-A): `RecordScope` in `verbs.go` and the config check both refuse scope partial with no skipped test, so the record cannot exist.
- The `## Regression check` heading could be stripped from the landing commit message by a commit cleanup mode (lens-code-correctness, unconfirmed): the grep found no cleanup option anywhere in the merge path, and a message passed as given keeps its `#` lines.
- Stale compare fields surviving a scope replacement: `RecordScope` replaces a record in place only before the baseline wait settles, which is before any compare has run, and the baseline and config paths build a fresh record.
- Comment, doc and stencil prose: `doc.go`, the spec paragraph and both stencils carry the verdict and drop or weaken no existing rule, and every new marker is filled non-empty at its only call sites.
