# Card 3 — seat-gate-fork-round

**Edit:**
- `internal/seatengine#Seat`
- `internal/seatengine#Table`
- `internal/seatengine#Table.Validate`
- `internal/seatengine#seatSpec`
- `internal/seatengine#Engine.startSeat`
- `internal/seatengine#Engine.probeSeat`
- `internal/seatengine/table_test.go`
- `internal/seatengine/prompt_test.go`
- `internal/seatengine/engine_test.go`
- `internal/seatengine/resume_test.go`
- `internal/seatengine/doc.go`

**Intent:**
Give the seat engine three generic seat capabilities a Burler round needs (decisions D2, D3 and D6): a per-advisor gate, per-seat fork authorization and the run's round number.

`Seat` gains `Gate shuttleengine.GateSpec`, an advisor's own gate, and `ForkSubagents bool`, which authorizes that seat's run to spawn in-session fork subagents.
`Table` gains `Round string`, the round every seat's run directory is named by; empty keeps today's naming.
`Table.Gate` stays the chair's gate.

`Table.Validate` refuses a non-empty `Gate` on the chair seat as a loud error naming the seat, since the chair's gate is the table's.
The input that reaches the refusal is a table whose chair seat carries a gate entry, with every other rule satisfied; a `TestTable_Validate` row reaches it.

`seatSpec` copies the seat's `ForkSubagents` and the table's `Round` onto the spec.
`Engine.startSeat` starts an advisor with its seat's `Gate` (empty for an ungated advisor, as every advisor is today) and the chair with the table's gate.
`Engine.probeSeat` probes an advisor with its seat's `Gate` and the chair with the table's gate, so a gated advisor's run is found as it was started.
A table that sets none of the new fields runs exactly as before, which keeps the Discussion seats table unchanged.

Tests:
- `table_test.go`: a `TestTable_Validate` row for the chair's seat gate, and a row accepting an advisor gate and fork authorization on any seat.
- `prompt_test.go`: `TestSeatPrompts_ValuesPromptsAndSpecs` asserts the spec carries the seat's `ForkSubagents` and the table's `Round`.
- `engine_test.go`: `TestEngine_Run_ChairEndStopsEverySeat` asserts an advisor with a seat gate is started with exactly that gate and an ungated advisor with none.
- `resume_test.go`: `TestEngine_ProbeAndResume_AttachToALiveChair` asserts an advisor with a seat gate is probed with it.

Document the three fields in the seat-model and table-rules sections of `internal/seatengine/doc.go`.

**ImpactSummary:** Three optional seat-table fields and one new validation rule; the existing Discussion table sets none of them and behaves as before.
