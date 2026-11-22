# Card 14 — goto-clears-pr-review

**Create:**
- `plan:internal/loomcli#prReviewEntryReset` -> `func prReviewEntryReset(reset func() error) func(target string) error`
- `plan:internal/loomcli#loomCLI.prReviewGotoAdmitted` -> `func (c *loomCLI) prReviewGotoAdmitted() func(target string) error`
- `plan:internal/loomcli#clearPRReviewOnGoto` -> `func clearPRReviewOnGoto(recordPath, lockPath string, stopLiveSession func(guid string) error, commit func() error) error`
- `plan:internal/loomcli#TestPRReviewEntryReset` -> `func TestPRReviewEntryReset(t *testing.T)`
- `plan:internal/loomcli#TestClearPRReviewOnGoto` -> `func TestClearPRReviewOnGoto(t *testing.T)`

**Edit:**
- `internal/shedengine#GotoRequest`
- `internal/shedengine#Goto`
- `internal/shedengine#RelaunchRequest`
- `internal/shedengine#Relaunch`
- `internal/shedengine#TestGoto_Guard`
- `internal/shedengine#TestRelaunch`
- `internal/shedverbs#Spec`
- `internal/shedverbs#RunGoto`
- `internal/loomcli#loomCLI.specFor`
- `internal/loomcli#loomCLI.loomConsumeAttempt`
- `internal/loomcli#loomCLI.goDriverDepsFor`

**Uses:**
- `internal/loomcli#loomCLI.gotoGuardEndingFixers`
- `internal/loomcli#loomCLI.stepStopRunner`
- `internal/loomcli#TestFixerEndingGuard`
- `plan:internal/pullrequest#ClearReviewSession`
- `internal/sheddriver#ConsumeTacklerAttempt`
- `internal/shedarbiter#ConsumeAttempt`
- `internal/loomcli/doc.go`
- `internal/shedengine/doc.go`
- `internal/shedverbs/doc.go`

**Intent:** D4: `lym loom goto` onto PR-Review clears the record's autonomy and session fields, as `settleSendBack` clears the session,
so the re-entry records the autonomy anew and starts a new session rather than resuming the recorded one.
The clear runs only once the move is written, so a refused or failed move leaves the record as it stood.

The engine hook:

- `GotoRequest` and `RelaunchRequest` gain `Admitted func(target string) error`, called with the request's `Target` once the move is written and still under the run lock; nil calls nothing.
- `Goto` calls it after its status write.
  `Relaunch` calls it after its write only when `Target` is non-empty and `Admit` did not report the move in effect, so a relaunch that moves nothing clears nothing.
  `ConsumeTacklerAttempt` and `shedarbiter.ConsumeAttempt` copy their base request, so the hook reaches both relaunches unchanged.
- An `Admitted` error comes back as the call's error with the move already written.
  Its text, built by the hook, names `lym loom goto --to PR-Review` as the way forward, which repeats the clear on the run the move left paused at PR-Review.
  The tackler's and the arbiter's relaunch have already marked their attempt applied, so their refused consume reaches the run's parent through the stop report, as every refused consume does,
  and the parent runs that goto.
- `Goto`'s comment, which says it sets no transient mark and commits nothing, names the one exception:
  the `Admitted` hook a recipe passes, which may reset and commit the records of the row the move entered, which the move restarts.
  `Relaunch`'s comment, which says it commits nothing, names the same exception.
- `shedverbs.Spec` gains `GotoAdmitted func(target string) error`, and `RunGoto` passes it as the request's `Admitted`, as it passes `GotoGuard`.

The loom side:

- `prReviewEntryReset(reset)` returns a hook that runs `reset` and returns its error when the target is `loomshed.NamePRReview`, and does nothing for any other target.
  `CheckGotoTarget` refuses a target past the current row before the move, so a move onto PR-Review always finds the run at or past it.
- `clearPRReviewOnGoto` reads the record; when it names a strand, it runs `stopLiveSession` on that strand's GUID before it clears the record with `pullrequest.ClearReviewSession`, then runs `commit`.
  A missing record clears nothing and commits nothing.
  `endSession` leaves the session live when the row's context was cancelled, as on a pause, so a run paused at PR-Review keeps its session; the stop keeps the re-entry from starting a second one beside it.
  A stop error is the hook's error and clears nothing.
- `prReviewGotoAdmitted` returns `prReviewEntryReset` over `clearPRReviewOnGoto`:
  the record at `shedrun.PullRequestRecord(c.location, shedrun.SelfRunID)` and its lock,
  `stopLiveSession` stopping the strand through `StopStrand` on the runner `c.stepStopRunner()` answers,
  and `commit` through `pullRequestRecordCommitter`, as `pr-continue` commits its write.
  `stopLiveSession` resolves the runner inside the closure at call time and never captures it when the hook is built:
  `goto` runs under `wireLightweight`, which leaves `c.reed` and `c.runner` nil,
  and `stepStopRunner` returns `c.runner` when `wire` built one and builds the pair's runner the way `wire` does otherwise.
  `StopStrand` marks the strand's shuttle run stopped when it still reads running,
  and removes the strand only when reed tracks it as removable: live, held for a relaunch, or an own-window strand with a dead pane.
  That is wider than a live-only stop, and wanted here: any strand the record names belongs to the session the clear discards,
  so the seam takes no separate liveness read.
  On this path `stepStopRunner` loads the reed and shuttle configs, which the lightweight `goto` otherwise never reads.
  A load error is a stop error, so the move stays written and the record keeps its fields,
  and the hook's text names the failed load to repair before the `goto` that repeats the clear.
  A receiver with no location returns nil.
- Every site that builds its guard through `gotoGuardEndingFixers` sets the hook beside it:
  `specFor`'s `spec.GotoAdmitted`, which `RunGoto` and the driver's `driverGoto` both read;
  `loomConsumeAttempt`'s arbiter relaunch; and the tackler relaunch in `goDriverDepsFor`.
  `shedverbs/doc.go`'s description of the spec's goto seams covers the new field.

Tests, tier0:

- `TestPRReviewEntryReset`: target PR-Review runs the reset once and returns its error; another target runs none.
- `TestClearPRReviewOnGoto`, over a temp record and lock: a record holding autonomy and session fields loses them, keeps `GatedHead`, and the commit stub runs once;
  a record naming a strand runs the stop stub once, on that GUID, before the clear;
  a stop error is returned, and the record keeps its fields and no commit runs;
  a record naming no strand runs no stop;
  a missing record runs no commit and returns nil; a commit error is returned.
- `TestGoto_Guard` gains rows: an admitting guard calls `Admitted` once with the target after the status holds the move, and a refusing guard calls none.
- `TestRelaunch` gains rows: a moving relaunch calls `Admitted` once with the target; a relaunch whose `Admit` reports the move in effect, one with an empty target and one whose `Admit` refuses call none.

**ImpactSummary:** One optional post-write hook on the two engine move requests and the verbs' spec, set only by loom's sites; only a move onto PR-Review gains a side effect, on the run's own pull request record, and every other recipe passes nil.
