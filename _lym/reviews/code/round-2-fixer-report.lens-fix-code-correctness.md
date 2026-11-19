Task: check the fix-code-correctness lens over the F15, F16 and F17 changes.

no findings

I ran each changed branch with one concrete input and traced each call's failure case.

- **F15, `settleUnlessRefused` (`internal/shedadapters/bouncer.go`):**
  - A CONVERGED verdict recovered on disk, with a done ask whose report holds findings, now returns Stuck through `rejudge`.
    The verdict is archived, so a retry judges fresh. `Commit` is not reached.
  - A Bouncer with no seat, or a seat that asks at round start, gets `heldBeforeJudgeAsk` = 0.
    Settle does nothing and `parentFindingsRefusal` returns "", so those paths behave as before.
  - An unreadable seed or ask record logs a warning and returns 0, so the round goes on without the parent and the seat never fails the round.
  - Settling before `routeDecisions` keeps the order `judgeCall` had, so an escalated round still settles its ask.
  - The re-judge after a refusal finds the held ask through `beforeJudgeAsk` and sends no second one.
  - The only callers of `settleUnlessRefused` are the three recovery paths and the harvest in `judgeCall`, and none depends on `judgeCall` having settled first.
- **F15 test, `TestBouncer_Call_RefusesARecoveredConvergedOverParentFindings`:**
  - With the defect in place, the check lives only in `judgeCall`.
  - The `entryHarvest` path then goes through `settleUnlessRefused`, finds no refusal, and returns Done with a commit, so the Stuck, archive and zero-commit assertions all fail.
- **F16, `renderParentReviewBlock`:**
  - A missing or unfilled fragment returns a `promptRenderError`, which `renderFailureReason` and `renderFailureCause` unwrap, so the judge degrades.
  - The prompt text is the same as before (`"\n\n" + block + "\n"` against the old constant).
  - The existing prompt assertions, on the heading and `lym loom parent-review await`, still hold.
  - `judgeParentBlock` has no remaining references.
- **F17, `runAgent`:**
  - `p.judgeStartedAt` is now set at the top of `runAgent`, which is the only place `parentAgreement` is reached, through `tableWithParent`.
  - `judge` and `passCommand` both call `runAgent`, so line mode and command mode are both capped.
  - Line mode's clock read moves later by a few steps.
- **F17 test, `TestDriftVerify_RunAgentCapsTheParentAskByTheRunsStart`:**
  - With `judgeStartedAt` left zero, the entry gets no limit and the ask's deadline would be `judgeParentNow` plus 120 minutes, so the equality against the 55-minute cap fails.
  - The fake `Run` evaluates the agreement entry's `Gate` inside the run, after `runAgent` has set the start, so the test covers the production order.

Further tool calls: none, because my context already held every changed file and the gate result.
