# Card 9 — seat-lead-amendment

**Edit:**
- `internal/seatengine#Table`
- `internal/seatengine#Engine.startSeat`
- `internal/seatengine#Prompts`
- `internal/seatengine/doc.go`
- `internal/seatengine#TestSeatPrompts_ValuesPromptsAndSpecs`
- `internal/seatengine#TestEngine_Run_ChairHoldDeferral`

**Intent:** D10's one gap in the seat engine: a fresh spawn's prompt amendment, appended to the lead's prompt only.

`Table` gains `Amendment string`: text appended verbatim to the lead's composed prompt when the table starts fresh, empty leaving the prompt byte-identical.
`Engine.startSeat` appends it to the lead's prompt after composition; a later seat's prompt never carries it.
`Prompts` appends it the same way, so a table builder's test sees what the lead will read.
`Engine.Probe` and `Engine.Resume` never read it: a probe's spec carries no prompt, and a resumed lead keeps the prompt it started with, so the amendment rides the fresh path only.
`doc.go`'s "Running a table fresh" section says so, and its notice-sender section states that a one-seat table queues no line and its lead's hold deferral never holds, since the deferral counts advisors and the sender carries advisor lines only.

Tests:
- `TestSeatPrompts_ValuesPromptsAndSpecs` gains a row: a table with `Amendment` set gives a lead prompt equal to the amendment-free prompt followed by the amendment, and an unchanged later-seat prompt.
- `TestEngine_Run_ChairHoldDeferral` gains a one-seat block of its own (a `t.Run` beside its advisor table, since its rows all act on an advisor): an `engineTable` of zero advisors runs fresh, the lead's started spec carries a hold deferral that reports false before and after the lead's session reads busy, and the lead's handle receives no `Send` before it is released done.

**ImpactSummary:** One added Table field read on the fresh path only; every existing table leaves it empty, so no prompt changes.
