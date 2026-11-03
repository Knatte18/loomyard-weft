# Fixer-report, code review round 1

Gate repair: the gate refused the round because the F1 comment in `internal/sheddriver/agentshuttle.go` broke one sentence across two lines; it is now two sentences, one per line.
Gate: `lym gate test ./internal/shuttleengine ./internal/sheddriver ./internal/shedarbiter ./internal/loomcli` was green (tier0, no failures) before that comment repair.
Some findings share a file, so several commits carry more than one finding id.

## Changes

- **F1** — fixed differently. Not made: setting `Spec.APIErrorRetries` on the gate fixer, conflict resolver, seat reviewers, burler, treadle, landing and webster specs.
  The callers there (`burlerreview.go`, `burlerfix.go`, `multillm.go`, landing, webster, seatengine) branch on `OutcomeTimeout` and `OutcomeDied` and have no api-error branch, so an opt-in would read as one more failure.
  The review's other option was taken: `internal/sheddriver/agentshuttle.go` states once, beside `SettlerAPIErrorRetries`, why only the judges opt in.
  The Goal names "any other role run under a deadline", so the parent still has to record the narrowing with `lym loom decision add` (see Deferred).
- **F2** — `string(OutcomeAPIError)` added to `unfinished` in `carryover.go` with its doc comment, plus a row in `TestStart_CarriesOverOperatorInstructions`.
  The twin prose in `internal/shuttleengine/doc.go` (carry-over paragraph) is updated too.
- **F3** — `api-error` added to the cause list at `contracts/orchrole/orch-role.md:135`.
  The twin list in `internal/loomcli/doc.go` is updated too.
- **F4** — `wantQuestion` field and a `strings.Contains` assertion in `TestTacklerCall_SpawnPath`; the api-error row sets `server_error`.
- **F5** — row "a later turn start and turn end clear the API error, so nothing is re-sent" in `TestWait_APIErrorRetry`.
  The events file already holds the START and STOP after the API error, so deleting the clear case makes the run re-send.
- **F6** — the review's second option: the `retryAPIError` doc comment states that a usage-limited session is re-sent and handed on like any other API error.
- **F7** — `apiErrorUnknownText` ("unknown error") is the fallback in `endOnAPIError` for both the log and `Result.APIError`; row "an API error with no text ends the run naming an unknown error".
- **F8** — a failed re-send goes through `run.failures.Fail("api-error re-send", ...)`, and a delivered one through `Recover`.
- **F9** — fixed differently. Not made: naming `retryAPIError` among the helpers that own the files check.
  After F13 the tripwire no longer audits `retryAPIError` (it reaches `allOutputFilesExist` through `classifyDeadlineExpiry`), so the entry says that in one sentence instead.
  `OutcomeAPIError` is added to the negative answers, and to `wait.go`'s file doc comment that the entry points at.
- **F10** — `docs/overview.md` names `api-error` and the `APIErrorRetries` re-send.
  Twin prose is updated too: the `internal/shuttleengine/doc.go` package doc, `wait.go:1` and `rundir.go` (the `finalize` comment).
- **F11** — rows in `TestWait_APIErrorRetry`: a covering pause (run held until the pause lifts), a failed send then a delivered one (asserts every failing send was attempted), earlier API errors seeded in the events file, and an API error before the prompt offset not counted.
- **F12** — fixed differently. Not made: an `api-error` row in `TestAttach_OutcomeDisposition`. That fixture's strand is tracked and live, so the row passes with the `isTerminalOutcome` case deleted, the same as the `unrecognized_outcome` row.
  An `api-error` entry is added to both outcome lists of `TestAttach_UntrackedTerminalRecord_RespawnEligibleRegardlessOfAge`, where a terminal outcome and an unrecognized one differ.
  For the shadow pairing, `TestLoopAgreesWithState_APIError` is a new small table test: the `TestWait_LogsSessionStateBesideItsClassification` drive compares a parser engine with one without, and only the parser engine ends a run api-error, so no row of that table can hold it.
- **F13** — `outcome := run.classifyDeadlineExpiry(OutcomeAPIError)` is computed once in `retryAPIError`.
  The hoist leaves no literal `OutcomeAPIError` return for `TestCompletionSignal_NegativeVerdictReturnSites`, so its `retryAPIError [OutcomeAPIError]` entry and the comment line naming it are removed.
  A new literal negative return in that spot still reads as an unaudited site and trips the test.

## Disputed

- **Self-check fix-proposal Finding 1** — ground `premise false`.
  The claim is that after the hoist "any later change to that return reads as zero sites" and a literal `endOnAPIError(OutcomeAPIError)` would pass the tripwire.
  The scan keys on a literal negative outcome inside a return statement: a later `return run.endOnAPIError(OutcomeAPIError)` is one site against zero audited, and the test fails with "a NEW negative-verdict return must consult allOutputFilesExist" (the failure the first gate run showed for the opposite count).
  The part of the finding about the PATTERN list is fixed (F9 above).
- **Self-check fix-proposal Finding 2** — ground `premise false`.
  "Any other turn end without every output file never ends a run" says nothing about the deadline, a dead pane or a stop; the sentence it extends already lists those.
  No text is wrong, so none is changed.
- **Self-check fix-proposal Finding 3** — ground `premise false`.
  `internal/logger/episode.go` says `Episode.Fail` logs "Warn at the first failure, Debug while it repeats, Info at recovery", so "logged once per episode" is what the code does.
  The first failure is a Warn where the old line was an Info; that is the review's own policy ("as `handleGatedBoundary` does").

## Self-check

- fix-code-correctness, LOW, attach row does not pin F12 — fixed with my own change (rows moved to the untracked and binding-cleared attach tests).
- fix-code-correctness, LOW, prompt-offset resume not pinned — fixed (the prompt-offset row).
- fix-code-correctness, LOW, failed-send row may never reach the failure branch — fixed with my own change: the row asserts every failing send was attempted. The Fail/Recover log levels stay unasserted; the episode's level policy is `logger`'s own.
- fix-consistency, LOW, three-outcome prose twins — fixed (`doc.go`, `wait.go:1`, `rundir.go`); the `shuttlecli` texts are unchanged: those verbs never set `APIErrorRetries`, so they never end `api-error`.
- fix-consistency, LOW, `loomcli` cause list — fixed.
- fix-consistency, LOW, PATTERN names a helper the tripwire no longer audits — fixed (F9 above).
- fix-proposal, MEDIUM, tripwire guard lost — disputed (premise false), PATTERN part fixed.
- fix-proposal, LOW, F10 wording — disputed (premise false).
- fix-proposal, LOW, F8 log level — disputed (premise false).
- fix-affected-set, LOW, `doc.go` carry-over twin — fixed.
- fix-affected-set, LOW, `doc.go` package-doc twin — fixed.
- fix-affected-set, LOW, `wait.go`/`rundir.go` twins — fixed.
- fix-affected-set, MEDIUM, `loomcli/doc.go` cause list — fixed.
- fix-affected-set, LOW, `shuttlecli` texts — no change (see above).
- Every fork returned a report.

## Deferred

- **F1** — the narrowing of the Goal's "any other role run under a deadline" to the judges needs a decision record from the run's parent (`lym loom decision add`).
  I may not message another session about the review, so the report carries it.
