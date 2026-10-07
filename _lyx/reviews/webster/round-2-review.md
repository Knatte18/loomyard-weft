---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: internal/burlerengine/engine_test.go:1067
    summary: no row covers a timed-out half whose stop fails, so the ErrHalfNotStopped return that keeps a retry from running beside a live half is untested
  - id: F2
    severity: LOW
    class: design
    location: internal/burlerengine/handoff.go:176
    summary: join stops a timed-out half but not one whose Wait returned an error, so its no-half-is-left-live claim in the join comment and doc.go does not hold for that half
  - id: F3
    severity: NIT
    class: consistency
    location: internal/burlerengine/handoff.go:186
    summary: the logger.Info call lacks the space after the first comma and fails gofmt
---

Round 2 reviews the branch's whole range, `git diff $(git merge-base main HEAD)..HEAD` (`product.parent` is `main`; the run has no `_lyx/loom/rework` rounds).
`go build ./...`, `go vet` and `go test` over the touched packages pass, `go test -race -count=3 ./internal/burlerengine/` passes, and `lyx loom lint-comments --base <merge-base>` reports ok.

### Focus answers

1. **Timed-out half stopped in `join`: yes.**
   The loop at `handoff.go:177` stops every half whose result is `OutcomeTimeout` through `stopHalf`, after the first `select` has stopped the other half when the deciding half failed.
   A failed stop in either place sets `stopErr`, which `join` returns before the outcome switch, wrapped as `ErrHalfNotStopped`.
   The `join` comment and the `doc.go` timeout sentences match this for a timeout, and `shuttleengine`'s `finalize` confirms that only a done run is cleaned up, so a timed-out strand stays live.
   The wording `no half is left live` is wider than the code in one case, raised as F2.
   The failed-stop path of the second call site has no test, raised as F1.
2. **A live attempt-1 half beside attempt 2: no path.**
   Attempt 2 starts only when attempt 1 returns a nil error with `OutcomeDied` or `OutcomeTimeout`.
   Every nil-error route through `join` to those outcomes has stopped the timed-out half (reviewer timeout, fixer timeout while the reviewer runs, fixer timeout after the handoff), and a died half's pane is not alive, so `RemoveStrandIfLive` would be a no-op for it.
   The marker is written only by the reviewer goroutine, which `join` has waited for on every nil-error path, and `Run` removes the marker before starting.
   A `notStartedResult` return stops the started reviewer first.
   An error return, `ErrHalfNotStopped` included, never reaches attempt 2, since `Call` does not retry on a runner error.
3. **Test rows: covered, with one gap and no standalone test added.**
   `TestEngine_Run_RoundFailureRules` gained the reviewer-timeout row and the fixer-timeout row now asserts both removed guids, both as rows of the existing table, and the budget row did not grow for them.
   The gap is the failed stop of a timed-out half (F1).
4. **Recipe comments: not a defect.**
   Each of the three `BurlerRound` rows states `No model/effort key: model choice lives in loom.yaml's review and fix keys only.` once, and `entries_burler.go` rejects `model` and `effort` through `configRejectUnknown`, with `entries_burler_test.go` pinning both keys.
   The second comment line, `No timeout_s key, for the same Env fallback reason as <Bouncer>`, matches the row's `timeout_s` handling.
5. **burler Long help: not a defect.**
   The paragraph is fixed-column text like the surrounding help string, which is user-facing output and not a Go comment, so the line-break pattern does not apply to it.
   Its content matches the engine: the fixer orients, waits for the accepted review, validates and fixes, and writes the fixer-report.

### [MEDIUM] No test row for a timed-out half that cannot be stopped

**Location:** `internal/burlerengine/engine_test.go:1067` (`TestEngine_Run_RoundFailureRules`)
**Issue:** The diff added the second `stopHalf` call site, the loop at `handoff.go:177`, so a timed-out half is stopped.
The behavior that protects the retry is that a failed stop there returns `ErrHalfNotStopped` instead of the `OutcomeTimeout` result, because `BurlerProducer.Call` would otherwise retry as attempt 2 beside a live half.
No row reaches it.
The existing `a half that cannot be stopped` row fails in the first `select` (a died reviewer, the fixer's stop fails), so it never reaches the loop, and the timeout rows run with a working remover.
A row is enough, since the table already carries `removeErr`, `wantErrIs` and the way-forward assertion (`PATTERN-test-economy`: a behavior the diff adds with no coverage).
**Fix:** Add one row to `TestEngine_Run_RoundFailureRules`: the reviewer is done with an approved review, the fixer's result is `OutcomeTimeout` with `waitForMarker`, `removeErr` is set, and the row wants `ErrHalfNotStopped` with the way forward naming `fixRole + "-guid"` and `wantRemoved` holding that guid.
The reviewer is accepted, so the first `select` stops nothing and the loop's stop is the failing call.

### [LOW] join leaves a half whose Wait errored unstopped, against its no-half-left-live claim

**Location:** `internal/burlerengine/handoff.go:176`, `internal/burlerengine/doc.go:100`
**Issue:** The loop stops a half only on `OutcomeTimeout`.
A half whose `Wait` returned an error (a liveness mechanism failure, a gate evaluation error, a fork-audit error) has an empty or unreliable result and is never stopped by `join`;
the other half is stopped, the error is returned, and `BurlerProducer.Call` archives the round's files through `failureExit` while that half's pane may still be live.
The `join` comment says `no half is left live` and `doc.go` says `join returns only after both waiting goroutines have returned, so no half is left live`, with the failed-stop half as the only stated exception.
That is the invariant round 1's F1 fixed for a timeout, left open for this class of end.
**Fix:** Stop a half whose `Wait` returned an error in the same loop as a timed-out one, so a failed stop returns `ErrHalfNotStopped` and an error end leaves the half stopped.
Add a row to `TestEngine_Run_RoundFailureRules` for a reviewer whose `Wait` errors (`halfScript.err`) asserting both guids removed and the wrapped shuttle error returned.
Keep the `join` comment and the `doc.go` sentences as they are, since they then hold.

### [NIT] logger.Info call missing a space after a comma

**Location:** `internal/burlerengine/handoff.go:186`
**Issue:** `logger.Info("burler: round halves joined","round", ...` fails `gofmt -l`; it is the only touched file that does.
**Fix:** Run `gofmt -w internal/burlerengine/handoff.go`.
