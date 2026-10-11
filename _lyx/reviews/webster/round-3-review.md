---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/battenshed/publishretry.go:261
    summary: publishRetryResume calls noticeDriverBusy without its new cause argument, so package battenshed does not compile and the Publish verify fails
---

### [BLOCKING] publishRetryResume calls noticeDriverBusy with the pre-merge signature

**Location:** `internal/battenshed/publishretry.go:261`

**Issue:** The Publish verify failure (`.lyx/verify/verify-6.log`) is a compile error: `not enough arguments in call to p.noticeDriverBusy; have (context.Context, *childWait); want (context.Context, *childWait, error)`.
I reproduced it with `go build ./internal/battenshed/` and `go vet ./internal/battenshed/`; both fail at the same line.
The cause is in the merge-in commit `0f61d0c13`: it brought in main's `86982fa43`, which changed `noticeDriverBusy` in `internal/battenshed/notice.go` to take a `cause error` and to quote the refusal's box text in the notice, and updated its one caller in `innerrun.go`.
The branch's card 10 (`publish-retry`) added a second caller, `publishRetryResume` in `publishretry.go`, written against the old two-argument form, and the merge did not touch it.
The whole `battenshed` package, and every test in it, including `innerrun_publishretry_test.go`, fails to build until this is fixed.

**Fix:** Pass the spawn error, `p.noticeDriverBusy(ctx, w, err)`, as `innerrun.go` does with `spawnErr`.
`err` is the `Spawn` error in scope there, already wrapped around `ErrChildDriverBusy`.
`TestInnerRun_PublishRetryRetriesRefusalsUntilTheyLandOrTheCapIsReached` asserts the notice by its fixed condition, which the new signature keeps as the notice key, so the test should hold unchanged; confirm it with `lyx gate test ./internal/battenshed`.

## Scope of this review

Beyond the compile break, I read the merge-in's `battenshed` diff and the branch's `battenshed` call sites of the changed notice seam.
I did not read the other cards' changes line by line, and I ran no tests, because the package does not build.
No other finding is raised.
