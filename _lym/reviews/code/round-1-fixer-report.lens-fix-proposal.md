```yaml
lens: fix-proposal
outcome: reported
```

Restated task: challenge the reviewer's proposals F1 to F5 as the fixer carried them out. The code changes survive; the landing summary left in sync only in part.

**Findings**

1. **LOW — `_lym/landing/summary.md`, intro line 8 and the "Tests" section.**
   - Line 8 still says "each wake that leaves the review open reads whether the session the call keeps is live". After F1 only a wake that read the status does, and a wake that read none never checks.
   - The "Tests" section says `TestPRReview_Wait` "gains two rows". F4 added a third, `AFailedLivenessReadKeepsWaitingAndLogsOnce`.
   - It also says the dead-session row halts on "the next wake". The row now halts only on a wake a poll interval later, which F3 forced.
   - The fixer edited only the "Where the check runs" bullet.

2. **LOW — F5's justification is false, though the change is sound.**
   - The review claims `endReview` is the only writer of `ReviewEnded`. `internal/landingshed/finalize_pr.go:565` also writes `ReviewEnded: true`.
   - That record is read at the PR-Review entry (`prreview.go:302`), which passes `keepsSession=false` or removes the record.
   - `judge` (`prreview.go:970-985`) returns Done or a halt for `ReviewEnded` under merged, closed and open alike, so a wait never carries on with `ReviewEnded` set.
   - So dropping `|| rec.ReviewEnded` breaks nothing. I built the case: a `ReviewEnded` wake where `stopLiveSession` returns `stopped=false` and the session is dead. It still ends in `judge` before the check runs.
   - The fixer should not repeat the "only writer" claim in the report.

**Proposals that held**
- **F1:** a wake that read no status skips the check, and `judge` still sees every ending first. The resume-after-stop path is unchanged.
- **F2:** warn-and-continue does not bring back the original hang. The `sessionLiveness` seam (`internal/loomcli/landingdeps.go:1056-1067`) returns `false, nil` for a strand absent from reed's table, so a removed strand still halts. Only a failure to read the table is swallowed.
- **F3 and F4:** the clock advance gives a polled wake, as the fixer's own arithmetic shows.

**Reads**
- `sed` of `prreview.go` lines 940-1060 plus a grep for `ReviewEnded`, to check that `judge` always ends on `ReviewEnded` (my context lacked `judge`).
- Grep for `SessionLive` to find where the seam is built.
- Grep for `SessionLive:` to confirm the production wiring in `sharedbootstrap.go`.
- `grep -A25 "func sessionLiveness"`, to see whether a removed strand returns an error or `false`.
