# Round 1 fixer-report

## Changes

- **F1** (fixed, own change in part): `endOnAPIError` in `internal/shuttleengine/apierrorretry.go` now calls `removeEndedOwnWindowStrand` (renamed from `removeDiedOwnWindowStrand` in `wait.go`, with its logs now reading "ended run") for every `OutcomeAPIError` end.
  The review proposed the call in the `watch.rejected` branch only; the self-check showed the retries-spent end leaves its strand live the same way, so both ends go through the one site.
  A strand `StopStrand` later finds untracked is a no-op, so the tackler and arbiter stops still hold.
  Rows with `ownWindow` and `wantRemoved` pin the removal for the rejected-model rows and the retries-spent row.
- **F2** (fixed as proposed): `doc.go`, `wait.go` (header and `Wait` doc) name the `ModelRejected` exit; `engine.go`'s `OutcomeAPIError` doc and `doc.go`'s died-capture sentence follow.
- **F3** (fixed as proposed): the tackler and arbiter hand-back text reads `result.APIError` alone; `tackler.go`'s doc and the `shedarbiter` and `sheddriver` package docs no longer claim spent re-sends.
- **F4** (fixed as proposed): `TestWait_APIErrorRetry` gains a rejected-model row with every output file on disk (done) and one with a covering pause (end held until it lifts).
- **F5** (fixed as proposed): `TestRunCmd_APIErrorEnvelope` in `internal/shuttlecli/cli_test.go` asserts `apiError` on a rejected-model run and its absence on a done run.

Commits, one per finding group: F3, F5, F4, F1 with F2 together (they share `wait.go` and `doc.go`), then the self-check follow-up.
`lym gate test` over `shuttleengine`, `shuttlecli`, `sheddriver` and `shedarbiter` is green.

## Self-check

- fix-code-correctness, LOW, the done row's STOP events line is not what ends the run: fixed, events trimmed to `START` with a comment naming the deadline.
- fix-code-correctness, LOW, retries-spent end leaves its strand live: fixed with my own change, see F1.
- fix-consistency, LOW, `engine.go` `OutcomeAPIError` doc stale: fixed.
- fix-consistency, LOW, `doc.go:76` names died runs only: fixed.
- fix-consistency, LOW, `shedarbiter/doc.go` and `sheddriver/doc.go` stale: fixed.
- fix-consistency, LOW, `apierrorretry.go` file header mentions only `APIErrorRetries`: disputed, premise false; its line 2 already states the at-once rejected-model end.
- fix-consistency, LOW, `apierrorretry.go` "as a died run's end does" cross-reference: fixed, the sentence is gone.
- fix-proposal, LOW, retries-spent end leaves its strand live: fixed, see F1.
- fix-proposal, LOW, anchored or window 0 strand stays after the end: fixed, `doc.go` names the bound.
- fix-affected-set, LOW, `engine.go:34`: fixed.
- fix-affected-set, LOW, `doc.go:121`: disputed, premise false; it stays true as written and `doc.go:123` states the rejected case.
- fix-affected-set, LOW, `shedarbiter/doc.go:15` and `sheddriver/doc.go:91`: fixed.

No fork returned no report.

## Disputed

- fix-consistency LOW on the `apierrorretry.go` file header: premise false, line 2 reads "or at once when the provider rejected the session's model".
- fix-affected-set LOW on `doc.go:121`: premise false, the sentence describes the retries path and is correct.

## Deferred

Nothing deferred.
