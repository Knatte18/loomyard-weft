---
verdict: APPROVED
---

No findings. Each round-2 focus question settles as follows.

- `internal/shuttleengine/gate_test.go` asserts `GateOutcome.EndedEarly` for every cause.
  - The send failure and the unrenderable re-prompt are rows of `TestGate_SendFailsMidLoop_EndsLoopWithAttemptsSoFar`.
  - The entry-wait expiry is a row of `TestGate_HoldsItsBoundaryWhileCovered`.
  - A session end or deadline before a re-prompt is asserted in `TestGate_DeadlineExpiresBetweenAttempts`.
  - A pane with no input box is asserted in three rows of `TestGate_RepromptOverPicker`.
  - `TestGate_EndedEarlyStaysEmpty` pins the empty value on a pass, a terminal failure and a failure whose entry spent its attempts.
- `internal/loomcli`: `TestWire_DarnRecipe` pins `--stop-on-fail` on the `darn-write` key's `VerifyCommand`.
  It pins `code-burler-fix` and `finalize` without the flag.
  `TestDarnTreeScope_ClassifiesAtEachCall` pins the same split on `VerifyTreeScope`, and an empty command stays empty.
- Budget: `TestVerifyGate_NoticesTheParentOnceItsBudgetIsSpent` drives `NewVerifyGate` with a 30 minute `Limit`, an injected clock and a recording `Notify`.
  `TestWire_DarnRecipe` pins `verify_budget_min: 45` as `45*time.Minute` with a non-nil `Notify`.
- Doc comments: `gateEndedEarly` and `finalizeAndRemoveDiedWindow` each carry exactly one doc comment above their own `func` line in `internal/shuttleengine/wait.go`.
- Rerun branches: `TestVerifyGate_DarnFormRunsTheFailedTestsAloneFirst` has rows for the dirty tree, timeout, runner crash and no-named-test reruns.
  `TestRunFailedTestsAlone_UnrenderableCallLeavesTheArrivalToTheFullCommand` covers the render error.
  `TestGateTest_Scenario` covers the build-blocked and doubly crashed `--stop-on-fail` cases.
  `TestVerifyBudget` covers the notice cut on a rune boundary and the exit-code fallback.
- Notice scope: `verifyBudgetNotice` names every failed run so far, latest first, which satisfies the board entry's "the gate's failures to date".

## Rejected

- The `-C names the run directory` line left the `gate test` long help.
  The `--dir`/`-C` flag usage still states it, so no rule is lost.
- A negative `verify_budget_min` reads as off without a refusal.
  The board entry asks for no validation, and the template documents 0 as off.
