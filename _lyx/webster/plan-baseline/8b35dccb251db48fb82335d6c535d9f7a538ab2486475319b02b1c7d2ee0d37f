# Card 6 — sheddriver-budget-handback

**Edit:**
- `internal/sheddriver#decideDrive`
- `internal/sheddriver#stopFromStatus`
- `internal/sheddriver#goDriver.reportFor`
- `internal/sheddriver#TacklerDeps`
- `internal/sheddriver#TacklerCall.resumePrecondition`
- `internal/sheddriver#TestDecideDrive`
- `internal/sheddriver#TestGoDriver_Settle`
- `internal/sheddriver#TestTacklerCall_VerdictOutcomes`
- `internal/sheddriver#tacklerFixture`
- `internal/sheddriver#tacklerFixture.deps`
- `internal/sheddriver#TestConsumeTacklerAttempt`
- `internal/sheddriver/doc.go`

**Create:**
- `internal/sheddriver#budgetExhausted`

**Intent:** D3's driver rule, D4's reading of the reason, and D2 at the tackler.

`budgetExhausted` is true for a stop whose key state is `blocked` and whose `Reason` starts with `shedengine.ReasonBounceBudgetExhausted`, or, for a stop built from the status, whose `Error` does.
`decideDrive` gets a hand-back row for it right after the paused row: ahead of the hand-over row, the `ResumeReturned` arbiter row and every tackler row, so the driver hands that halt back also after a tackler resume returned to it, also for a stop read from the status, and also when the on-disk build changed, never handing it over or resuming it.
Its rule text is `a bounce-budget-exhausted halt: only the parent's resume lifts it`.
Bound: the row only hands back; it never resumes a run.

`stopFromStatus` fills `Reason` from the status's `pause_reason` for a paused status, so the paused row reads the reason from the stop or from the status.
The paused row asks for the friction note whenever the reason is empty, from either source, which a paused status an older binary wrote leaves; `reportFor` labels such a stop's reason `paused with no recorded reason`.

`TacklerDeps` gains `GotoGuard shedengine.GotoGuard`; `TacklerCall.resumePrecondition` runs it after `CheckGotoTarget` on a verdict's goto, so a verdict naming a refused row is rejected as an invalid resume with the guard's refusal (or error) text; nil admits as today.

`doc.go`'s rule table adds the row and the paused-reason reading.

Tests. `TestDecideDrive` adds rows: a budget halt from the envelope hands back; the same from the status (`stopFromStatus` of a blocked status whose error carries the prefix) hands back; a budget halt with `BuildChanged` and with `ResumeReturned` still hands back; a status-derived paused stop with a `pause_reason` asks for no note and one without asks for it (the existing "status-derived paused stop carries none" row changes to carry a reason).
`TestGoDriver_Settle` adds rows: a blocked budget-halt envelope hands back with no tackler, arbiter or hand-over call; a paused stop with an empty reason writes a report holding `paused with no recorded reason`; a tackler resume whose `ConsumeTackler` returns a guard refusal on an unmoved run hands back with a report holding that refusal text; a step envelope that refused the arbiter attempt with a guard refusal in its `error` hands back with a report holding that text, which `reportFor` takes from `Error` when `Reason` is empty.
`TestTacklerCall_VerdictOutcomes` adds two rows whose fixture carries a `GotoGuard` in a new `tacklerFixture` field that `deps()` copies: a refusing guard rejects the resume verdict naming the refusal, and an erroring guard rejects it naming the error.
`TestConsumeTacklerAttempt` adds a row whose base `RelaunchRequest` carries a refusing `Guard`: the consume returns the refusal, writes no applied mark and leaves the status file unchanged.

**ImpactSummary:** One new rule-table row, paused-reason reading and an optional tackler seam; loom and batten driver wiring compiles unchanged.

**Commit:** 6: sheddriver: a budget halt goes back to the parent, the paused rule reads the recorded reason, the tackler runs the goto guard
