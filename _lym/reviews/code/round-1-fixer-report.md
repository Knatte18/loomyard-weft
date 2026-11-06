```yaml
round: 1
review: _lym/reviews/code/round-1-review.md
gate: lym gate test ./internal/shuttleengine — tier0 passed, no failures
```

## Changes

Commits: `99de0a8d` (F1, F2, F3), `0aa4b9ca` (F4), `fcfd4944` (F5), `26b7ec45` (self-check findings).
A commit holds more than one finding where findings share a file.

- **F1, fixed as proposed.**
  `loadSkillTurn` no longer skips an open turn at a bound: `loadTurnOpenBound` is gone.
  A turn whose start was read and has no end is awaited until its end, the pane's death or the run's deadline.
  The deadline is `run.deadline` moved later by the hold before the turn (`heldDeadline`).
  At the deadline it returns a new `*loadDeadlineError`, and `abandonLoad` ends the start through `classifyDeadlineExpiry(OutcomeTimeout)` with no prompt typed.
  A turn whose start was never read still skips at the timeout.
  The "skipped at the open bound" wording is out of `doc.go`, `spec.go` and the `loadturn.go` comments.
  The open-turn row was reshaped to run to the deadline and assert no prompt is typed, a `timeout` outcome and at least an hour of virtual time.
- **F2, fixed as proposed.** Rows in `TestStartup_LoadTurnAPIError`:
  - a turn start read during the back-off, so no second load is typed and the prompt follows;
  - a busy retype, so the load is typed after the doubled back-off;
  - the run and persisted prompt offsets equal the events file size, asserted on every row that starts.
  The fakes gained `Carries`, `LoadSendErrs` and a raw late line.
- **F3, fixed as proposed.** Rows for the 5 minute ceiling, a retype failing with another error (died), an interactive run, a failing notifier, a rejected model, and a hold-over log assertion on the died and rejected returns.
  Row 7 is reshaped, as the review allowed.
- **F4, fixed as proposed.**
  `doc.go` (deadline, load timing, open-turn behavior), the `Run.deadline`, `Runner.notifier` and `SetNotifier` comments, and the `SkillLoadTimeout` comment now match the code.
- **F5, fixed as proposed.**
  `PATTERN-daemon-wakeups.md` names the `pollInterval` wake and its floor.
  `PATTERN-reed-panes.md` gives the API-error hold a Rule line and an Enforcement line naming `TestStartup_LoadTurnAPIError`.

## Self-check

Four forks returned.
- fix-affected-set, no findings.
- fix-consistency LOW: the `loadSkillsThenPrompt` doc comment did not name the deadline failure. Fixed as proposed.
- fix-consistency LOW: `doc.go` "a skipped skill never fails or hangs it" read against the new timeout. Fixed as proposed, with the sentence naming the one case that fails.
- fix-consistency LOW: the `OutcomeTimeout` doc did not name a load turn open at the deadline. Fixed as proposed in `engine.go`.
- fix-consistency LOW: the `loadSkillTurn` first sentence said only "timeout passes". Fixed as proposed.
- fix-code-correctness LOW: the offset assertions cannot fail on a change inside `readLoadTurn`. Disputed, premise false.
- fix-code-correctness minor: the ceiling row hard-coded `30*time.Second`. Fixed as proposed, now `apiErrorRetryBackoff`.
- fix-proposal MEDIUM: an interrupted open load turn would hold to the deadline. Fixed with my own change: the open branch asks the engine's `SessionProber.TurnStartInterrupt` and skips the turn at once as a timed-out one, since an interrupt writes no turn end. New row, `doc.go` sentence.
- fix-proposal MEDIUM: a slow healthy turn now fails a start whose `Spec.Timeout` is short. Fixed differently: the review asked for exactly this end, so the behavior stays, and the `SkillLoadTimeout`, `OutcomeTimeout` and `loadSkillsThenPrompt` comments now say the start fails.
- fix-proposal LOW: the carry row depends on the startup probe taking one `Status` call. Fixed as proposed, by stating the coupling in the `carriedTurn` comment.
- fix-proposal LOW: the Rule line said "never typed" beside an attach bound. Fixed as proposed, reworded to the start's own prompt line.

## Disputed

- **fix-code-correctness LOW, offset assertions, ground: premise false.**
  Dropping `run.offset = newOffset` in `readLoadTurn` is no defect: `loadSkillsThenPrompt` sets `run.offset` and `PromptOffset` from `eventsSize(...)` at `run.go:831-837` right before the prompt, and that assignment is the guarantee.
  The assertion fails if that assignment is removed, moved before the hold or taken from a size read before the re-typed turns.

## Deferred

- **F6, deferred to the run's parent, as the review directs.**
  The model-rejected end (`loadModelRejectedError`, `abandonLoad`, `Result.APIError`) and the parent notice line (`noticeLoadHold`) stay in the code untouched.
  The parent decides whether they stay and records them with `lym loom decision add`.
  Their rows in `TestStartup_LoadTurnAPIError` and the dedicated model-rejected test stay with them.

## Gate repair

The gate refused to run tests because a comment I added in `startup_test.go` (the prompt offset assertions) was wrapped mid-sentence.
It now breaks at the sentence boundary, one sentence per line.
No other change.

## Notes

- The interrupted-turn skip logs the existing `skill skipped` warning with cause `timeout`, since `settleLoadTurn` names that cause for any `ended == false`.
- `doc.go:124` has a pre-existing run-together line (`...changes the model.// RunState.NotifiedOffset...`) that this round did not touch.
