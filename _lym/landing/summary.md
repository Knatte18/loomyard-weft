# Settlers re-send API-error turns, then hand the stop on naming the error

When the provider's API ends a Claude turn on an error, Claude fires `StopFailure` instead of `Stop`.
claudeengine already reads that line as an API-error turn-end signal, but `ParseEvents` yields no event for it, so shuttle's wait loop never saw a turn end.
An arbiter or tackler whose turn died this way sat idle until its deadline.
It wrote no verdict and the parent was never told.
The run's Go driver kept its lock all that time, so the parent's `lym loom resume` was refused as `driver_not_parked`.
On the `cs` hub this held `report-significant-digits` at Drift-Await for 26 minutes.

## What changes

**Shuttle** (`internal/shuttleengine`)

- `Spec.APIErrorRetries` sets how many times a run re-sends a turn the provider's API ended while an output file is still missing.
  The new `apierrorretry.go` reads the session signals each tick from the prompt offset.
  An API-error turn end stays pending until a turn start, turn end or session start follows it.
  The n-th one is re-sent with one `[lym]` line through the gated send, after `30s × 2^(n-1)`.
- The API-error turn end after the last re-send ends the run with the new `OutcomeAPIError`, and `Result.APIError` holds the provider's error text.
  This exit goes through `classifyDeadlineExpiry`, so a run whose output files are all on disk still ends done (Completion Signal Invariant).
  A pending API error that finds every output file present ends the run done at once.
- `api-error` counts as a terminal record outcome for attach, and the session-state shadow pairs it with `idle-stalled`.
- With `APIErrorRetries` at zero, nothing changes: every other spec still waits such a turn end out to its deadline.

**Settlers** (`internal/sheddriver`, `internal/shedarbiter`)

- The arbiter and tackler specs set `APIErrorRetries` to `sheddriver.SettlerAPIErrorRetries` (3).
- When a run ends `OutcomeAPIError`, the call stops its strand and hands on (arbiter) or hands back (tackler) with the new cause `api-error`.
  The reason names the error, and so does the question: `HandBackQuestionCause` puts the error after the cause.
  The driver's hand-back then parks the run and sends the parent the question notice.
  That notice names the run, its stop report, the settler and the error, so a dead settler no longer holds a stop silently, and the parent's resume finds the run parked.

## Design decisions

- **Detection reads session signals, not a new event kind.**
  Making `StopFailure` an `Event` would have changed every `ParseEvents` consumer: the orch watcher, gatefixer, activity and session reads.
  The retry instead keeps its own signal cursor, which is provider-neutral through `SessionSignalParser`, and only runs for a spec that opts in.
- **A new outcome, not `died` or `timeout`.**
  The session is alive and its deadline has not passed, so both existing outcomes would misreport it.
  `OutcomeAPIError` lets each caller name the cause.
  Callers that only test for `OutcomeDone` treat it as one more failure.
- **The retry budget counts API-error turn ends since the prompt offset**, not re-sends made by this process.
  A re-attach therefore resumes the count from the run's own events file, not from zero.
- **Only the two settlers opt in.**
  These are the roles that hold a stop alone under a deadline.
  Other autonomous roles keep their current behaviour.
- The Completion Signal tripwire now also scans `apierrorretry.go`, with `OutcomeAPIError` added as a negative marker and its one return site audited.

## Tests

- `TestWait_APIErrorRetry` drives Wait over a fake session with these rows:
  - a re-send after the back-off lets the agent finish;
  - repeated API errors end the run `api-error` naming the error, with the back-off doubling between re-sends;
  - a spec with no retries waits to its deadline;
  - an API error with the output file on disk ends the run done with no re-send.
- New rows in `TestCall_VerdictOutcomes` and `TestTacklerCall_SpawnPath` cover the `api-error` hand-on and hand-back.
  The arbiter row also checks that the question names the error.
- `TestSpec` and `TestTacklerSpec` check the retries on both settler specs.
