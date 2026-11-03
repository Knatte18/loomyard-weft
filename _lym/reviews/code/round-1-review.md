---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/pullrequestmode.go:106
    summary: A non-transient failure of the parent-rule read in Finalize reads as no rule, so Finalize closes the recorded pull request and lands by a direct push that the rule refuses, which is the goal's bug.
    fix: In internal/landingshed/pullrequestmode.go make resolveWithParentRule take a flag (or the found record) saying a pull request record exists, and when the rule read fails with a non-transient, non-token error and a record exists, return an error or halt instead of logging a Warn and reading as no rule. In internal/landingshed/finalize_pr.go:137 read the record before calling resolveWithParentRule and pass that flag, and halt with githubHaltReason("read GitHub's rules for the parent branch", err) and LandingStopPrefix so the way forward is lym loom resume. Keep the soft Warn path only for a run with no recorded pull request. Add a Finalize row with a record, parentMode off and a RequiresPullRequest 404 asserting a halt, no Close call and the record kept.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/prreview.go:684
    summary: An open pull request whose record carries ReviewEnded and a dirty tree makes the PR-Review wait poll forever, since the session is skipped for an ended record and judge's open case only keeps waiting; the old closed path ended Stuck.
    fix: In the StateOpen case of judge in internal/landingshed/prreview.go, before the rec.Continue check add a case for rec.ReviewEnded that returns done() on a clean tree and stuck(p.dirtyEndReason(rec.Number, dirty)) on a dirty one, as the StateClosed case does. Add a wait row in internal/landingshed/prreview_wait_test.go with seed("h1", withContinue, withReviewEnded) and an open status for the clean case (Done, no Edit call, record unchanged) and one with dirty paths (Stuck naming the files and lym loom resume).
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/landingshed/pullrequestmode.go:381
    summary: The soft-fail of a token failure on the rule read, and the cancelled-context error of that read, have no test row, so removing the tokenFailure guard would turn every token-less hub into an error stop with no test failing.
    fix: In TestPRReview_Call (internal/landingshed/prreview_test.go, near the rule rows) add a parentMode off row whose RequiresPullRequest script returns githubclient.ErrTokenUnresolvable and expects Done, no error and wantCalls of RequiresPullRequest main; add a row with a context cancelled before the read expecting the cancel error. Add the token row to TestFinalize_PullRequest (internal/landingshed/finalize_pr_test.go) beside the transient row, expecting the local landing to go on.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/landingshed/prreview_test.go:156
    summary: The skip conditions of resolveWithParentRule, an empty task branch and an origin with a missing fact, have no row asserting the rules are not read.
    fix: In TestPRReview_Call add two parentMode off rows, one with an empty task branch and one with a local origin, each with an empty wantCalls and outcome Done, so dropping the taskHasCommits or checkOrigin early return fails a test.
---

### [MEDIUM] Finalize can still land by a direct push when the rule read fails after PR-Review opened the pull request

**Location:** internal/landingshed/pullrequestmode.go:106, internal/landingshed/finalize_pr.go:137
**Issue:** PR-Review opens a pull request because GitHub's rules require one, so the run holds a record. If Finalize's own read of the rules then fails with a non-transient error such as a 403 or 404, `resolveWithParentRule` reads no rule, the decision is no, and `dropRecordedPullRequest` closes the pull request, removes the record and lets Finalize squash locally and push to a branch that requires a pull request. That is the failure the goal names. The goal says lym never chooses a direct push to such a branch, and the read has to fail safe when a record shows the rule applied earlier.

### [MEDIUM] A reviewed-ended open pull request can hold the PR-Review wait forever

**Location:** internal/landingshed/prreview.go:684
**Issue:** The diff keeps the pull request open at the end of the review, so a PR-Review re-entry can find an open pull request whose record has `ReviewEnded` and `Continue`. `openPullRequest` then waits without a session, and `judge`'s open case only tests `Continue`: on a dirty tree it returns keep waiting, and nobody is there to clean it. The closed-pull-request path this replaces ended Stuck with `dirtyEndReason`. The open case needs the same verdict.

### [MEDIUM] The token-failure and cancelled-context paths of the rule read are untested

**Location:** internal/landingshed/pullrequestmode.go:381
**Issue:** `resolveWithParentRule` checks `tokenFailure` before `transientFailure` because the HTTP client wraps token errors as transient. The only error rows use a 404 and a deadline error, so neither the token soft-fail nor the cancelled-context error is pinned. This is a behavior the diff adds with no coverage.

### [LOW] No row asserts the rules are skipped for an empty task branch or an unusable origin

**Location:** internal/landingshed/prreview_test.go:156
**Issue:** The summary claims the rules are read only for a task branch with commits and an origin a pull request could be opened on. The existing local-origin row has the field on, so it halts before the read and proves nothing about the skip.

## Rejected

- Code-correctness F3 (a pull request closed on GitHub for another reason is reopened and merged when its record carries the ending): the summary states this as the accepted design, and `ReviewEnded` is the only evidence the goal's "Finalize reopens it" can rest on.
- Code-tests F1 (no row for re-entry on an open pull request with `ReviewEnded`): merged into F2, whose fix adds those rows.
- Handler check of the goal's three Expected bullets: the open pull request is kept, a closed one with the ending is reopened and merged, and the rules read feeds `pull_request` mode. No change beyond them was found, and `PATTERN.md`, the stencil, `docs/overview.md` and the package docs match; the stencil keeps its other rules. The affected-set lens found nothing left stale.
