# Card 6 — seat-runner-gate-once

**Edit:**
- `internal/testkit/shedfake#SeatRunner`
- `internal/testkit/shedfake#SeatRunner.Run`
- `internal/testkit/shedfake#SeatRunner.Resume`
- `internal/testkit/shedfake#TestSeatRunner_LastEntryRepeatsAndFnsOverride`
- `internal/shedrecipe#TestMultiLLMEntry_ComposedTable`

**Intent:** Give the fake seat runner the gate-once contract `shedfake.Shuttle` carries for a gated run, so a row whose gate rides the chair, as Discussion-Write's does from card 7, fails or passes its gate inside the fake (D1, D9).
`SeatRunner` gains `GateAttempts` and `GateReason` fields, stamped onto the `GateOutcome` the fake builds, as `Shuttle`'s are.
`Run` and `Resume`, after answering `RunFn`, `ResumeFn` or the scripted entry, evaluate `table.Gate` once when the gate is non-empty, the chair's outcome is done and the answered chair result carries no `Gate` outcome of its own, stamping a `GateOutcome` onto the chair result and returning a gate closure's error.
Bound: a consumer whose `RunFn` or `ResumeFn` evaluates the gate itself returns the `GateOutcome` it built, and the fake then never evaluates the gate a second time; a table with no gate, a chair that is not done, and a scripted result already carrying a `GateOutcome` behave as today.
`TestSeatRunner_LastEntryRepeatsAndFnsOverride` gains rows: a gated table whose closure fails, stamping a failed outcome with the fields' attempts and reason; one whose closure passes; one whose `RunFn` returns its own `GateOutcome`, leaving the closure uncalled.
`TestMultiLLMEntry_ComposedTable` builds a table with a `plan` gate and scripts a done chair, so the fake would now run the real plan gate over a test env with no plan; its scripted chair result carries a passing `GateOutcome`, keeping it a table-shape test with the gate unevaluated.
The fake's other consumers in `shedadapters` and `shedrecipe` script tables with no gate, or results that already carry a `GateOutcome`, so they keep their behaviour.

**ImpactSummary:** One test-kit contract widened behind a no-gate-outcome condition; of the fake's consumers only `TestMultiLLMEntry_ComposedTable` scripts a gated table with a gate-free done chair, and it is adjusted here, with no production code touched.
