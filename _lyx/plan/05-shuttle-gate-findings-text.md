# Card 5 — shuttle-gate-findings-text

**Edit:**
- `internal/shuttleengine#GateOutcome`
- `internal/shuttleengine#Run.evaluateGate`
- `internal/shuttleengine#TestGateList_FinalReports`
- `internal/shuttleengine/doc.go`

**Intent:** D4's shuttle side: a gate outcome that did not pass carries the failing entry's findings text, because finalize removes the run directory, and the findings file in it, before the producer reads the outcome.

`GateOutcome` gains `Findings string`: the `GateResult.Findings` text of the last failing entry of the evaluation, empty when the outcome passed.
`FindingsPath`'s doc keeps that the path is diagnostic only and dead once `Wait` returns, and now points at `Findings` as the text a producer keeps.
`Reason` keeps its terminal-only meaning, unchanged.
`Run.evaluateGate` records the failing result's `Findings` beside `findingsPath` in the loop and sets `outcome.Findings` where it sets `outcome.FindingsPath`, when the outcome did not pass.
The change touches the gate outcome only, per `PATTERN-shuttle-provider-seam`.

`TestGateList_FinalReports`'s row "exhaustion without pass_on_cap finalizes failed", which already asserts `result.Gate.FindingsPath`, also asserts `result.Gate.Findings` is `"bad"`, the text `scriptedGate` returns for a failing call.
The assignment sits in the same `!outcome.Passed` block that sets `FindingsPath`, so no passed-outcome row is added: a row whose let-through entry runs no closure could not fail on it.
`doc.go`'s gate paragraph says a failed outcome carries the failing entry's findings text on `GateOutcome.Findings`, beside the terminal reason it already names.

**ImpactSummary:** one additive struct field and one assignment in the gate evaluation; no caller changes until card 7 reads the field.
