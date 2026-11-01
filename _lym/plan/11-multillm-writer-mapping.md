# Card 11 — multillm-writer-mapping

**Create:**
- `internal/shedadapters#MultiLLMProducer.writerExit`
- `internal/shedadapters#TestMultiLLMProducer_MapsAWriterThatEndedTheStep`

**Edit:**
- `internal/shedadapters#MultiLLMProducer`
- `internal/shedadapters#MultiLLMProducer.mapOutcome`
- `internal/shedadapters/doc.go`

**Uses:**
- `internal/shedadapters#TestMultiLLMProducer_MapsTheLeadResult`
- `internal/shedadapters#multiLLMTable`

**Intent:** Map a writer seat that ended the step onto the producer contract (D6), so a writer's own failure halts the run with its part committed and a reason naming the writer, instead of an `ErrAgentDied` naming the stopped chair.

`mapOutcome` reads the seat results before the chair's: the first `SeatResult` with `Writer` and `EndedStep` set is mapped by `writerExit`, and the chair's outcome then decides nothing.
A required advisor's end keeps today's mapping through the chair.
`writerExit`, with the seat's name as both seat and part, and `p.name` as the row:
- a non-empty `StartError` returns an error wrapping `shuttleengine.ErrNotStarted` carrying the start error;
- an outcome of died, or a wait error, returns an error wrapping `shuttleengine.ErrAgentDied` naming the writer seat;
- a timeout returns `shedengine.Stuck`, its pointer the seat's first output path, with the reason "writer seat <name> of part <name> timed out";
- done with a failed gate returns Stuck with the reason naming the seat and part followed by `gateFailedReason(gate, seat.GateFindingsPath)`, which keeps the findings in that file, and the gate's attempts and spans on the pointer;
- an Accept error returns Stuck with the reason naming the seat, its part and the Accept error;
- a writer lost at resume with outputs missing (no strand, no start error, no outcome) returns Stuck with the reason naming the seat, its part and that its outputs were missing at resume;
- any other end, such as an outcome shuttle does not name, returns an error naming the writer seat and the outcome, as the chair's unrecognized-outcome branch does.
Every Stuck reason ends with the way forward: re-run the row with `lym loom resume`, which starts every part over unless the chair is still live.
The Stuck pointer is always non-empty, so the commit decorator commits the part directories; a cancelled context returns the context error before any of this, as the chair's mapping does.
The two error cases are classified as the chair's are today: `internal/shedtransient` retries `ErrNotStarted`, and `ErrAgentDied` only after a recent spawn refusal; otherwise the died error fails the run, and its text names the way forward, re-running the row with `lym loom resume`.
The `MultiLLMProducer` doc comment, which says the advisors' results are never judged, is reworded: an advisor's result is logged only, and a writer that ended the step decides it.

`TestMultiLLMProducer_MapsAWriterThatEndedTheStep` (tier0, over `multiLLMTable` with one writer seat and `shedfake.SeatRunner`'s scripted results) has one row per case above asserting the outcome, the pointer path, the reason text or the wrapped error, and a row where the chair also timed out and the writer's mapping still decides; a findings-file row asserts the findings land in the seat's `GateFindingsPath`.
`TestMultiLLMProducer_MapsTheLeadResult` stays as is: a result with no writer maps as today.
`doc.go` gains the writer mapping.

**ImpactSummary:** mapOutcome gains one branch taken only for a writer seat's step end; every advisor and chair mapping is unchanged.
