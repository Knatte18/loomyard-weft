# Card 3 — gate-entry-reprompt

**Edit:**
- `internal/shuttleengine#GateEntry`
- `internal/shuttleengine#Run.handleGatedBoundary`
- `internal/shuttleengine#gateRepromptText`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/gate_test.go`

**Uses:**
- `internal/shuttleengine#Run.sendWithin`
- `internal/shuttleengine#validateSendText`

**Intent:** Let a gate entry supply its own re-prompt line (decision D2).
`GateEntry` gains the optional field `Reprompt func(findingsPath string) string`: when non-nil, the wait loop sends its return value after a failed attempt with budget remaining, in place of the default `gateRepromptText` line; nil keeps the default, so no other gated producer changes.
In `handleGatedBoundary`, build the re-prompt text from the failing entry's `Reprompt` when set, else from `gateRepromptText`, and pass it to `sendWithin` as today.
The text is validated where every send is, by `validateSendText` inside `sendWithin`: a renderer returning several lines, an empty or a whitespace-only line fails the send, which ends the loop with the attempts spent so far, exactly as any other re-prompt send failure does; no new path is added.
State this in the field doc, in the `gateRepromptText` doc (it is the default) and in the gate paragraph of `doc.go`.

Tests, in `gate_test.go`: turn `TestGate_FailsOnceThenPassesOnNextTurn` into a two-row table, the default line and an entry whose `Reprompt` returns one custom line naming the findings path, asserting the sent text is exactly what the renderer returned.
Add a row to `TestGate_SendFailsMidLoop_EndsLoopWithAttemptsSoFar`: a renderer returning two lines, with no reed send error, ends the loop with the gate failed, zero attempts and no `SendText` call.

**ImpactSummary:** One optional field on an exported struct and one read site in the wait loop; every existing caller leaves it nil.
