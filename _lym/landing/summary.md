# Re-send a judge's turn after an API error, then hand the stop on

A judge session (arbiter or tackler) whose turn ends on a provider API error used to sit idle until its deadline.
The stop it held reached no one, and the parent's `lym loom resume` was refused with `driver_not_parked` while the driver waited on the dead session.

Now `shuttleengine` reads that turn end, re-sends the turn after a back-off, and ends the run `OutcomeAPIError` once the re-sends are spent.
The arbiter and tackler calls turn that outcome into a hand-on or hand-back with cause `api-error`, and the question to the parent names the error text.

## Changes

- `shuttleengine`: new `Spec.APIErrorRetries`.
  `Run.retryAPIError` (`apierrorretry.go`) runs on each `Wait` tick and reads the session signals itself, since an API-error turn end is no event the run loop classifies.
  It re-sends one line once `apiErrorRetryBackoff` (30 s) times 2^(n-1) has passed since the n-th API-error turn end was read.
  The turn end past the last re-send ends the run `OutcomeAPIError`, and `Result.APIError` carries the error text.
- A turn start, turn end or session start read after an API-error turn end clears it, so a recovered session is not re-sent again.
  A pause that covers the run holds the re-send.
- Done wins: when every output file exists, the run ends done through `classifyDeadlineExpiry`, as the completion signal invariant requires.
  `attach` and the session-state shadow count API errors from the prompt offset.
- `sheddriver`: `CauseAPIError` (`api-error`) and `SettlerAPIErrorRetries` (3).
  `HandBackQuestionCause` puts the error text into the hand-back question.
- `shedarbiter` and the tackler call map `OutcomeAPIError` to a hand-on or hand-back with that cause, and stop the run.
- Prompts, `contracts/orchrole/orch-role.md`, `docs/overview.md` and `pattern/PATTERN-shuttle-provider-seam.md` name the new cause among the fault cases the orch investigates.

## Design decisions

- Only the arbiter and tackler opt in.
  Any other role's caller reads every outcome but done as one more failure, so a re-send there would only delay it.
  A role that wants the behaviour sets `APIErrorRetries` and handles `OutcomeAPIError`.
- The back-off doubles per API-error turn end, so a passing server error has time to clear and a settler keeps most of its deadline.
- A provider usage-limit stop is re-sent and handed on like any other API error.
- A failed re-send is logged once per episode and tried again on a later tick, bounded by the run deadline.
- An engine without `SessionSignalParser` and an unreadable events file change nothing.

## To check by hand

- No run against a live provider error was made.
  Trigger a `StopFailure` in a real arbiter session and confirm the re-send, then the hand-on naming the error, then a `lym loom resume` that is not refused.
- The parent is told through the existing hand-on and hand-back path only.
  There is no separate creel notice, so confirm the orch's stop report names the failed settler and the stop.
- Findings left open on the change:
  - `api-error-roles-under-deadline-not-opted-in`: roles other than the arbiter and tackler that run under a deadline do not set `APIErrorRetries`, in `internal/sheddriver/agentshuttle.go`.
  - `orch-role-contract-omits-api-error-cause`: check that `contracts/orchrole/orch-role.md` names `api-error` wherever it lists the hand-on causes.
  - `pending-clear-unpinned`: no test pins that a turn start, turn end or session start clears a pending API error, in `internal/shuttleengine/apierrorretry.go`.
  - `tackler-api-error-question-unasserted`: no test asserts that the tackler's hand-back question names the API error, in `internal/sheddriver/tackler_test.go`.
  - `unfinished-omits-api-error`: check whether the unfinished-run listing and classification in `internal/shuttleengine` should include `OutcomeAPIError`.
