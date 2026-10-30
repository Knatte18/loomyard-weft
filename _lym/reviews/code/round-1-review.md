---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/loomcli/landingdeps.go:336
    summary: the TaskHasCommits closure that wires the git check has no test, so a swapped from and to or a wrong ref passes every test
    fix: Add a test beside the existing loomcli landing-deps integration tests that builds a real task pair and asserts pullRequestDeps(...).TaskHasCommits() returns false for a task branch equal to its parent and true after one commit on the task branch; add a row where the parent ref does not resolve and assert a non-nil error.
  - id: F2
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/landingshed/prreview.go:84
    summary: neither row has a case where the TaskHasCommits seam returns an error
    fix: In TestPRReview_Call add a row whose TaskHasCommits returns an error and assert Call returns an error naming the read of the task branch's commits with no push and no GitHub call; in TestFinalize_PullRequest add the same row (Finalize reaches errorStop at finalize_pr.go:121) and assert the error stop and that nothing is merged.
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/landingshed/finalize_pr.go:81
    summary: no Finalize test feeds ErrTokenRejected, or one wrapped in a url.Error, through githubFailure, so the token-before-transient ordering is unpinned for Finalize
    fix: In TestFinalize_PullRequest add a row beside the existing ErrTokenUnresolvable row whose status call returns a url.Error wrapping githubclient.ErrTokenRejected, and assert a halt that names the token rather than a transient re-step error.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/loomcli/landingdeps.go:337
    summary: TaskHasCommits answers a boolean through CommitLog, which reads CommitDetail for every commit on the branch and discards the subject lines
    fix: Expose a fabricengine method that reports whether the task HEAD holds a commit the ref lacks, built on the existing CommitsNotIn result without CommitDetail, and call it from the TaskHasCommits closure instead of CommitLog.
---

### [MEDIUM] The git wiring of TaskHasCommits is untested

**Location:** internal/loomcli/landingdeps.go:336
**Issue:** The closure carries D2, the requirement that git decides the branch is empty. The diff's tests set the seam by hand or list it as intentionally nil in `wiring_seams_test.go`, so no test runs the closure over a real pair. A swapped `from` and `to`, a wrong ref or a dropped error would pass every test.

### [MEDIUM] The TaskHasCommits error path has no test

**Location:** internal/landingshed/prreview.go:84
**Issue:** Both rows return an error when the seam fails (`prreview.go:84`, `finalize_pr.go:121`). Every fixture returns `(!taskEmpty, nil)`, so that behavior the diff adds is uncovered.

### [LOW] Finalize's rejected-token ordering is unpinned

**Location:** internal/landingshed/finalize_pr.go:81
**Issue:** PR-Review's rows cover a rejected token inside a `url.Error`. Finalize's tests cover only the unresolvable token and the 404 text, so reverting Finalize's `tokenFailure` to the old check would pass. D5 names both rows.

### [LOW] CommitLog reads every commit to answer a boolean

**Location:** internal/loomcli/landingdeps.go:337
**Issue:** `CommitLog` resolves a `CommitDetail` per commit (`internal/fabricengine/parenttips.go:152`), so a long task branch pays one read per commit on every PR-Review entry and every Finalize pass for a yes or no.

## Rejected

- Finalize calls `TaskHasCommits` with no nil check (code-correctness F2): the real wiring and `envkit` set it together with `ParentMode`, `battencli` is loom-only and lands no branch, and Finalize's `EntryField` and `Origin` seams are used the same way, so no reachable path is left unset.
- No Finalize row for the 401, 403 and 422 halt text (code-tests 2): `githubHaltReason` is one function shared by both rows and `TestPRReview_Call` pins every split, so a Finalize copy of each row would duplicate coverage.
- The transport error and the row are tested only by composition (code-tests 5): `TestAuthRT_401Handling` pins the `%w` wrap and the PR-Review row pins the classifier order, which together cover the chain.
