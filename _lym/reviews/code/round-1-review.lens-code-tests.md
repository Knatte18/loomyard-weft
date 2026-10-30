```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the diff adds or changes the test row that pins it, or none.

**Behaviour → pinning test**
- **Mode resolver, D2–D4, nine combinations, origins, halts:** `TestResolvePullRequest` (`ModeAndFieldCombinations`, `Origins`), `pullrequestmode_test.go:15,19,48`.
- **Row decision paths (no, unreadable parent beam.yaml, field `on` with a local origin):** `TestPRReview_Call`, `prreview_test.go:136-159`.
- **Open or find, edit on re-entry, record survives a continue flag, `ClosedForContinue` re-entry:** `prreview_test.go:164-196`.
- **Push and GitHub failures (non-transient halt, transient error, token wrapped in a URL error, 404):** `prreview_test.go:204-229`.
- **Nil seam and cancelled context:** `prreview_test.go:282,315`.
- **Session start, reuse, resume by id, start failures, registration failure, cancel leaves it running, stop on Stuck and error, covered-wake stop and resume:** `TestPRReview_Session`, `prreview_session_test.go:145-273`.
- **Wait verdicts for merged (clean, dirty, missing commits, other head), closed, `ClosedForContinue`, continue, approval, older-head approval, comment after change request:** `TestPRReview_Wait`, `prreview_wait_test.go:187-292`.
- **Session stopped before the re-read, resumed only if stopped:** `prreview_wait_test.go:261,270`.
- **D8 gate (dirty waits, moved head gated and pushed, failure mails once per head, mail failure logged, no session logs, head or tree changed during the gate discards the verdict):** `prreview_wait_test.go:296-370`.
- **Transient poll warns once, token halt, 403 halt:** `prreview_wait_test.go:379-397`.
- **Wake interval doubling to a minute and falling back to a second:** `prreview_wait_test.go:401`.
- **Covering pause (no gate, push or GitHub call; at entry and at a wake):** `prreview_wait_test.go:424,448`.
- **Immediate poll on the continue flag, extra Status call on a dirty tree:** `prreview_wait_test.go:460,473`.
- **Wait mark set, re-set after the gate, cleared:** `prreview_wait_test.go:487`.
- **Finalize through the pull request (all paths from resolver halt to method refusal, pending retry, transient, token, push failure, fast-forward failure):** `TestFinalize_PullRequest`, `finalize_pr_test.go:77-165`.
- **Teardown close (open closed with comment, merged or closed left, no record, gone worktree warns, close or status failure warns, bad origin warns):** `TestCloseOpenPullRequest`, `teardown_test.go:1205-1254`.
- **Landing deps wiring and drift guard:** `TestLandingDeps_EveryFieldPopulated`, `landingdeps_test.go:44`.
- **Session seams (live, register, errors):** `TestPRSessionSeams`, `landingdeps_test.go:378`.
- **No parent lock on any path:** `seam_enforcement_test.go`, +6 lines (not opened).
- **Quiet check counts a child status running at PR-Review:** `wire_seams_test.go` (+187 lines).
- **`pr_reviewer` default and malformed value:** `landingshed/config_test.go:66,278`.
- **`ParseOwnerRepo` new shapes:** `parseownerrepo_test.go`.
- **`DenyPublish` and the git push and gh denies:** `settings_test.go`, `rows_integration_test.go`, `ghguard_test.go`.
- **`pr-continue` verb:** `commitrecords_test.go` matched a grep for it; I did not read it.
- **Recipes and registry:** `darnrecipe_test.go`, `pawlrecipe_test.go`, `entries_prreview_test.go`.
- **Board `pull_request` field:** `boardengine/task_test.go`.
- **beam.yaml `pull_request` key:** `dobbyengine/config_test.go`.
- **Retired keys:** the `loomengine`, `darnengine` and `configreg` test changes. I confirmed only their existence from the diff stat.

**Findings**

1. LOW, class `consistency`, `internal/landingshed/prreview.go:640-651` and `prreview_wait_test.go:240-245`.
   - Summary: no test row for an open pull request with the continue flag set on a clean tree whose head is not the gated head.
   - Detail: that case must keep waiting without calling `Close`. The nearest rows are the open-at-gated-head case that closes (`:240`), the dirty-tree case (`:245`), and the closed-pull-request ungated case (`:221`). If `!gated` were dropped from the `!clean || !gated` guard, the dirty row would still pass and only the closed-state row would be affected, so the open-state ungated branch is unpinned. I read the test names only, not the row bodies.

2. LOW, class `consistency`, `internal/landingshed/prreview.go:202-212`.
   - Summary: I found no row for a failing `RemoveRecord` or `ReadRecord` on the `ClosedForContinue` re-entry path. Those are error paths, not trust-boundary inputs, so I judge the gap minor. The rows I could see cover only the kept-record and new-pull-request cases (`prreview_test.go:188,196`).

I found no other behaviour in the diff without a pinning test.

**Tool calls**
1. Grepped test names and subtests across the landingshed, pairteardown and loomcli test files; the inherited context held only the production code and the diff stat.
2. Grepped for `pr-continue` and `DenyPublish` test coverage, plus the diff stat for stencils, recipes and `pr_reviewer` tests; the inherited context lacked the test-file contents.
