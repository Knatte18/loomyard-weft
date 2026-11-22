```yaml
lens: fix-code-correctness
outcome: reported
```

Findings from the lens fix-code-correctness check; no caller of a changed signature or behaviour is broken.

**MEDIUM: `AutonomyContinueWithNoStandingSeamReturnsTheMissingSeamError` (`internal/landingshed/prreview_wait_test.go`) passes with `judgeParentAnswer`'s own seam check removed.**
- The review asked for a nil `PRReviewStanding` row for `judgeParentAnswer`. `checkAutonomySeams` is also called at `prsession.go:76` and `prsession.go:92`, before the wait loop, and the fixer's own run shows the error arrives before the wait sets its mark.
- The earlier session-start check therefore returns the "PRReviewStanding is not set" error first. Deleting the call at `prreview.go:972` would leave the row green.
- The row pins the wiring error as a whole, not the `judgeParentAnswer` path. It also needed the `tt.wantErrText == ""` exemption on the wait-marks assertion.
- The erroring-standing row (`errStandingRead`) does reach `judgeParentAnswer` and is sound.

**LOW: `handBackNotify` is tested, but its use at the `HandBack.Notify` field (`internal/loomcli/drivewiring.go`) is not pinned.**
- `TestHaltNoticeSeams_CarryTheRunsAutonomy` calls `handBackNotify(pair)` directly.
- Reverting the assignment `Notify: handBackNotify(location)` to the unstamped `orchcli.NotifyParent` fails no test.
- F3 named `drivewiring.go HandBack.Notify` as the seam to pin, so the gap is partly open. The `newPrePushCheck(...).notify` row is sound.

**LOW: `TestGotoWiring_HandsTheGotoPathsThePRReviewResetHook` (`internal/loomcli/gotowiring_test.go`) only asserts the hook is non-nil.**
- It never calls the hook with `loomshed.NamePRReview`, so it does not show that the hook clears autonomy and session. F4 asked for that call.
- It does fail if `Admitted` is dropped from either path, since `prReviewGotoAdmitted` returns nil only when `location` is nil. This is a coverage shortfall against the finding's text, not a wrong result.

**Checked and holding:**
- **F1:** `ClearReviewSession` has a single caller, `clearPRReviewOnGoto`. The test row's held record has `ParentReviewAsked` true and would fail without the new line.
- **`relaunchRequest`:** it reproduces both inline requests field for field, with the same guard and runID at both call sites.
- **`WithRunAutonomy`:** the dobby `Post` stamps with the run slug and signs with `geom.Slug`, so the dobby's scope is kept. The new beamcli test asserts `From` for the scoped and unscoped dobby, and would fail unstamped.
- **Batten and loomcli stamp rows:** they would fail with no `; autonomy:` suffix.
- **`TestGoto_Guard`:** the new `admitErr` row sees the written move and the returned error, and asserts the move stays written.

Tool calls (1 of 6):
- `grep checkAutonomySeams internal/landingshed/*.go`: to find the callers of `checkAutonomySeams`, which my context lacked.
