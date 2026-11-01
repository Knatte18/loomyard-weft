# Card 9 — seat-writer-kind

**Create:**
- `internal/seatengine#writerChairDirectiveBlock`
- `internal/seatengine#writerDirectiveBlock`
- `internal/seatengine#writerDoneNotice`
- `internal/seatengine#Seat.endsStep`
- `internal/seatengine#Table.hasWriters`
- `internal/seatengine#TestEngine_Run_WriterSeat`

**Edit:**
- `internal/seatengine#Seat`
- `internal/seatengine#SeatResult`
- `internal/seatengine#Result`
- `internal/seatengine#Table`
- `internal/seatengine#Table.Validate`
- `internal/seatengine#chairDirectiveBlock`
- `internal/seatengine#validateSeatRoles`
- `internal/seatengine#seatRun`
- `internal/seatengine#newSeatRun`
- `internal/seatengine#seatRun.watch`
- `internal/seatengine#seatRun.result`
- `internal/seatengine#Engine.Run`
- `internal/seatengine#seatSpec`
- `internal/seatengine#stepEnder`
- `internal/seatengine#stepEnder.end`
- `internal/seatengine#stepEnder.fired`
- `internal/seatengine#TestTable_Validate`
- `internal/seatengine#TestSeatPrompts_ValuesPromptsAndSpecs`
- `internal/seatengine#TestEngine_Run_RequiredAdvisorEndsTheStep`
- `internal/seatengine/doc.go`

**Uses:**
- `internal/seatengine#fakeShuttle`
- `internal/seatengine#engineFixture`
- `internal/seatengine#seatedStencils`

**Intent:** Give the seat engine a writer seat kind (D5): a writing assistant that writes a product of its own the chair joins.

`Seat` gains three fields, each with a doc comment:
- `Writer bool` marks a writer seat; the lead is never one.
- `Timeout time.Duration` bounds that seat's run; zero takes the table's `Timeout`, so every existing table keeps its timing. The `Parts-Plan-Write` chair sets it to outlast its writers.
- `GateFindingsPath string` is where the producer keeps a writer's failed gate findings; the engine only carries it.

`SeatResult` gains `Writer bool`, `Outputs []string` (the seat's absolute output paths) and `GateFindingsPath string`, which `seatRun.result` fills from the seat; `seatRun` keeps its absolute outputs, which `newSeatRun` and the not-started and lost runs set.

`Seat.endsStep` reports `Required || Writer`; every place that today asks `seat.Required` to decide whether an end ends the step asks it instead (`seatRun.watch`, a start failure in `Engine.Run`), so a writer is required by kind: every end its gate and Accept do not pass ends the step, a failed start included, with no line queued.
`validateSeatRoles` admits an Accept hook on a seat that ends the step, and refuses `Writer` on the lead with "the lead is marked a writer, but only a seat after the lead can be".
`Table.Validate` adds, each naming the seat and the way forward:
- a table holding both a writer and a non-writer seat after the lead is refused: a table holds advisors only or writers only;
- the lead of a table that `hasWriters` must include `seat-directive-writer-chair` (`writerChairDirectiveBlock`) in place of `seat-directive-chair`, which every advisor table's lead still needs;
- a writer seat's stencil must include `seat-directive-writer` (`writerDirectiveBlock`).

`seatRun.watch`: a writer that ended done with its gate passed and its Accept passed queues `writerDoneNotice` to the chair, "Writer <name> finished its part and keeps answering you; its outputs stand: <outputs>", so a chair waiting on its parts is woken; a required advisor still queues nothing.
`Engine.Run` queues the advisor up line only for an advisor; a writer gets no up line, since its done line is what the chair waits for.
`seatSpec` sets `QuietHold` for every seat after the lead except a required advisor, so a writer waiting on the chair sends the parent no held-turn notice, and sets `Timeout` from the seat's own when non-zero.
A writer keeps its pane after done, as every seat after the lead does.

`TestTable_Validate` gains rows: a writer table whose lead includes `seat-directive-writer-chair` and whose writers include `seat-directive-writer` is accepted; a mixed table, a writer table whose lead includes only `seat-directive-chair`, a writer whose stencil lacks `seat-directive-writer`, and a writer lead are each refused; a writer with an Accept hook is accepted.
`seatedStencils` already seeds the shipped stencils, which hold the two directives, and the new rows write their test stencils with `writeStencil`.
`TestSeatPrompts_ValuesPromptsAndSpecs` gains a writer seat: its spec has `QuietHold` and `KeepPane` true and a seat `Timeout` overriding the table's.
`TestEngine_Run_RequiredAdvisorEndsTheStep` already has one row per end of a required advisor (died, timeout, wait error, failed gate, Accept error, failed start, not found at resume); it gains two writer rows, a failed gate (the `seatRun.watch` site) and a failed start (the `Engine.Run` site), each with a writer seat in place of the required advisor and asserting `EndedStep`, `Writer` and `Outputs` on the writer's result and nothing queued; the died, timeout, wait-error and Accept-error ends reach the same `Seat.endsStep` predicate at `seatRun.watch` and stay covered by the advisor rows, and the end at resume is card 10's.
`TestEngine_Run_WriterSeat` (tier0, over `fakeShuttle`) covers only what that table does not reach: a writer ending done with its gate passed and Accept passing queues the done line, and no up line is queued for it at start.
The `Table` doc comment's `Timeout` field and the `Result` doc comment's `Advisors` field are reworded: a seat's own timeout overrides the table's, and `Advisors` holds one entry per seat after the lead, advisor or writer.
The `Seat.Required` and `Seat.Accept` doc comments are reworded so Accept belongs to a seat that ends the step, a required advisor or a writer, and the `Table.Validate` doc comment's lead-directive rule says a writer table's lead includes `seat-directive-writer-chair` and an advisor table's lead `seat-directive-chair`.
`doc.go` gains a `# Writers` section stating these rules, and its existing sentences that writers falsify are reworded for writer seats: the lead-include sentence ("The lead's stencil of a table of more than one seat includes seat-directive-chair; …") says a writer table's lead includes `seat-directive-writer-chair`, the up-line sentence ("Once each advisor's start ends, the chair is told in one line …") says a writer gets no up line, and the step-end sentence ("Only a required advisor can end the step before the chair, …") says a writer can end the step before the chair too.
The other advisor-only comments are reworded the same way: the `Table.Validate` comment's "Accept belongs to a Required advisor", the `validateSeatRoles` comment's "Accept only on a Required advisor", the `Table` `Seats` field's "then the named advisors", the `chairDirectiveBlock` comment's rule that a lead of more than one seat includes `seat-directive-chair`, and `doc.go`'s "every later seat is an advisor with a unique valid name".
The `stepEnder` type and its `end` and `fired` methods are reworded the same way: their comments name the seat that ends the step, a required advisor or a writer.
So are the `SeatResult` comment's `EndedStep` line, which holds for a writer too, the `newSeatRun` comment, whose seat that ends the step on ender is a required advisor or a writer, and the `seatSpec` comment, which says every seat after the lead keeps its pane and holds its turn end quietly unless it is a required advisor.

**ImpactSummary:** Adds a seat kind and two seat fields read in the engine's validation, watch, start and spec; advisor tables and tables without a seat timeout behave as before.
