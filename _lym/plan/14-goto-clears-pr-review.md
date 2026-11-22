# Card 14 — goto-clears-pr-review

**Create:**
- `plan:internal/loomcli#prReviewResettingGuard` -> `func prReviewResettingGuard(inner shedengine.GotoGuard, atOrPastPRReview func() (bool, error), reset func() error) shedengine.GotoGuard`
- `plan:internal/loomcli#clearPRReviewOnGoto` -> `func clearPRReviewOnGoto(recordPath, lockPath string, commit func() error) error`
- `plan:internal/loomcli#TestPRReviewResettingGuard` -> `func TestPRReviewResettingGuard(t *testing.T)`
- `plan:internal/loomcli#TestClearPRReviewOnGoto` -> `func TestClearPRReviewOnGoto(t *testing.T)`

**Edit:**
- `internal/loomcli#loomCLI.gotoGuardEndingFixers`
- `internal/shedengine#GotoGuard`

**Uses:**
- `internal/loomcli#fixerEndingGuard`
- `internal/loomcli#TestFixerEndingGuard`
- `plan:internal/pullrequest#ClearReviewSession`
- `internal/loomcli/doc.go`
- `internal/shedengine/doc.go`

**Intent:** D4: `lym loom goto` onto PR-Review clears the record's autonomy and session fields, as `settleSendBack` clears the session,
so the re-entry records the autonomy anew and starts a new session rather than resuming the recorded one.
Goto admits the clear only on a run halted at PR-Review or later, never on one PR-Review still holds.

- `prReviewResettingGuard` runs `inner`; when `inner` admits, the target is `loomshed.NamePRReview` and `atOrPastPRReview` answers true, it runs `reset` and returns its error.
  A refusal, an inner error, any other target and a run halted before PR-Review run no reset; an error of `atOrPastPRReview` is the guard's error.
- `clearPRReviewOnGoto` clears the record with `pullrequest.ClearReviewSession` and then runs `commit`; a missing record clears nothing and commits nothing.
- `gotoGuardEndingFixers`, through which every goto that moves a run builds its guard, wraps its result in `prReviewResettingGuard`:
  `atOrPastPRReview` reads the run's status at `c.shedPaths.StatusPath` and answers whether its current row is PR-Review or a row after it in `c.routing.Producers`;
  `reset` is `clearPRReviewOnGoto` over `shedrun.PullRequestRecord(c.location, shedrun.SelfRunID)` and its lock, committing with `pullRequestRecordCommitter` as `pr-continue` commits its write.
  A receiver with no location resets nothing.
  The arbiter's and the tackler's bare guards only pre-check a target; their moves go through `gotoGuardEndingFixers`, at `arbiterconsume.go` and `ConsumeTackler`.
- `shedengine.Goto` takes the run lock before the guard runs, so a run whose driver holds PR-Review is refused before the clear.
- `GotoGuard`'s comment, which says a guard writes no file, names the one exception:
  once it admits, a guard may clear the records of the row the move enters, which the move restarts.

Tests, tier0:

- `TestPRReviewResettingGuard`, a table in `TestFixerEndingGuard`'s shape:
  an admitting inner with target PR-Review on a run at or past it resets once and admits;
  the same on a run halted before PR-Review resets nothing and admits;
  another target resets nothing;
  a refusing inner and an erroring inner reset nothing and return their refusal and error;
  a reset error and an `atOrPastPRReview` error are the guard's error.
- `TestClearPRReviewOnGoto`, over a temp record and lock: a record holding autonomy and session fields loses them, keeps `GatedHead`, and the commit stub runs once;
  a missing record runs no commit and returns nil; a commit error is returned.

**ImpactSummary:** One wrapper on the one guard builder every moving goto uses, and one sentence on the guard type's comment; only a goto onto PR-Review of a run at or past it gains a side effect, on the run's own pull request record.
