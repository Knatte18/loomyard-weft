```yaml
lens: code-tests
outcome: reported
```

**Per-behaviour list (behaviour → pinning test)**

1. `Verdict` and `renderVerdict` rendering:
   - Pinned by `TestVerdict` in `internal/driftengine/verdict_test.go`. The rows cover no record, scope off, waived, not run, passed, drifted, and accepted drift by caller, operator and agent, including the unruled suffix and the omitted Baseline line when `BaselineHead` is empty.
   - No row for scope full with a skip list, or for an unknown `Acceptance.By`. Both are minor.
   - No row for the non-`ErrNotExist` stat-error branch.
2. `Verdict` creates no directory when no record exists: pinned by `NoRecordCreatesNoDirectory`.
3. `recordCompare` after the compare in `pass`:
   - A pass is pinned by `requireCompareRecorded(..., true)` in the passing-compare row.
   - A failing compare that ends Stuck is pinned by `requireCompareRecorded(..., false)` in the final-compare row.
   - The `ComparedAt` value is pinned. The `BaselineHead` and `ComparedHead` values are not separately pinned, see F1.
4. `recordCompare` in `runAgent`'s Done case:
   - Pinned by the agent-Done row. The `pass` path has already written `false`, so dropping the agent record leaves `false` and the row fails.
   - No row for the interrupted-live-agent re-entry, which reaches `runAgent` with `told` built from the capture record. It is the same code path, so this is minor.
5. `recordCompare`'s `Head` error path and the `updateScope` error path: none. Minor.
6. `withSection` on the Finalize landing commit message:
   - Pinned by `RegressionCheckFollowsTheBody` in `TestFinalize_MergeOptionsCarriesComposedMessage`. It asserts the title, body, verdict and credit order.
   - An empty verdict is pinned by the existing nil-seam rows.
   - `withSection`'s empty-body branch (`section` returned alone): none, see F4.
7. Finalize's failing seam is an error before the merge: pinned by `regression_check_unreadable`.
8. Finalize's GitHub merge body: none, see F2.
9. PR-Review body on a first publish (`Create`): pinned by `RegressionCheckFollowsTheDescriptionBodyBeforeTheCredit`. A failing seam gives an error and no GitHub call: pinned by `FailingCreditOrRegressionReadIsAnErrorWithNoPullRequestCall`, which covers `DriftVerdict` and `EnginesRun`.
10. PR-Review body on a re-run refresh of an open pull request: none, see F3.
11. PR-Review session prompt, `sessionSpec`:
    - The verdict reaches the prompt: pinned by `PromptCarriesTheRegressionCheck`.
    - The empty-verdict fallback to `noDriftVerdict` is pinned by the `noDriftVerdict` assertion in the existing prompt row.
    - A `DriftVerdict` error in `sessionSpec`: none, see F5.
12. `DescribeTable` and `drift_verdict`: pinned by `TestDescribeSpec_DriftVerdict`, with a told verdict and with an empty one. `DriftVerdict` stays optional, so `TestDescribeSpec_EmptyFieldIsError` needs no row.
13. `Deps.driftVerdict` nil seam: pinned by the nil-seam Finalize and PR-Review rows.
14. Production wiring:
    - `landingDeps` filling `DriftVerdict` has no test, see F6.
    - The `DescribeTable` closure in `wiring.go` reading the verdict has no test, see F6.
15. Stencil prose: exempt.

**Findings**

- **F1, LOW, design.** `internal/driftengine/verify_test.go:247-253`: `requireCompareRecorded` pins both `BaselineHead` and `ComparedHead` to `"aaa"`, because the fixture's capture head and HEAD are the same. Swapping the two arguments in `recordCompare` (`internal/driftengine/verify.go:285`) or passing `told.BaselineHead` where HEAD belongs would still pass. Evidence: the helper's `want` values and `verify.go:187` and `verify.go:545`. Fix: give the fixture a capture head that differs from its HEAD in the passing and agent-Done rows.
- **F2, MEDIUM, scope.** `internal/landingshed/finalize.go:205`, `finalize.go:228`: `summary.Body` is mutated, so the verdict also reaches the PR-path `bodyWithCredit(summary.Body, credit)` handed to `landUnderLock`. The diff's Finalize test asserts only `MergeOptions.Message`, which is the landing commit message. The doc and summary claim the GitHub merge message carries the section, and no row asserts it. Fix: add a row to the Finalize pull-request merge test with `DriftVerdict` set, asserting the merge's title and message on the fake API carry the section before the credit.
- **F3, LOW, scope.** `internal/landingshed/prreview.go:240`: the body is computed for both Create and the re-run refresh, and the doc and spec claim the refresh path. Only the Create row has the verdict. Fix: add a refresh row to `TestPRReview_Call` with `DriftVerdict` set, asserting the title and body update call carries the section once, without doubling.
- **F4, LOW, scope.** `internal/landingshed/finalize_pr.go:93-103`: `withSection`'s whitespace-only body branch has no row. Fix: add a row with an empty-body description and a verdict, asserting the body is the section alone.
- **F5, LOW, scope.** `internal/landingshed/prsession.go:125-128`: the error return from `driftVerdict()` in `sessionSpec` has no row. The `Call` error test covers `openPullRequest`, not the session. Fix: add a `TestPRReview_Session` row whose `DriftVerdict` seam errors, asserting no session start.
- **F6, MEDIUM, scope.** `internal/loomcli/landingdeps.go:236-237` and `internal/loomcli/wiring.go:816-821`: no test pins that production `landingDeps` fills `DriftVerdict`, or that the `DescribeTable` closure passes `Verdict` through. `Landing.DriftVerdict` was added to `nilLegal` (`internal/testkit/envkit/envkit.go:88`). `nilLegal` is what `NilSeams` skips, so the nil-seam guard no longer catches an unwired `DriftVerdict`. The `deps.go` doc says "its drift guard keeps it filled", but nothing asserts that. Fix: add a direct check in `internal/loomcli/landingdeps_test.go`, in the style of the existing direct checks at `internal/loomcli/wiring_test.go:937`, that `landingDeps` returns a non-nil `DriftVerdict` rendering a seeded scope record. Add another that `DescribeTable` carries the verdict into the prompt.

Tool calls:
1. `grep -rn "nilLegal" internal --include=*.go` plus a grep for tests calling `landingDeps(`: to learn what `nilLegal` and `NilSeams` guard and whether a test pins the production seam. My context lacked the guard's use sites.
