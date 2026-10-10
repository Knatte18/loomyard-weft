# Card 9 — shedadapters-multillm

**Create:**
- `plan:internal/shedadapters#SeatRunner` -> `type SeatRunner interface`
- `plan:internal/shedadapters#multiLLMEngineLabel` -> `const multiLLMEngineLabel = "seats"`
- `plan:internal/shedadapters#MultiLLMProducer` -> `type MultiLLMProducer struct`
- `plan:internal/shedadapters#NewMultiLLMProducer` -> `func NewMultiLLMProducer(name string, table seatengine.Table, seats SeatRunner, now func() time.Time) *MultiLLMProducer`
- `plan:internal/shedadapters#MultiLLMProducer.Call` -> `func (p *MultiLLMProducer) Call(ctx context.Context) (shedengine.Outcome, shedengine.OutputPointer, error)`
- `plan:internal/shedadapters#MultiLLMProducer.mapOutcome` -> `func (p *MultiLLMProducer) mapOutcome(ctx context.Context, result seatengine.Result) (shedengine.Outcome, shedengine.OutputPointer, error)`
- `internal/shedadapters/multillm_test.go`

**Edit:**
- `internal/shedadapters/doc.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shedadapters#SingleLLMProducer.Call`
- `internal/shedadapters#SingleLLMProducer.mapOutcome`
- `internal/shedadapters#archiveStaleOutputs`
- `internal/shedadapters#entryErr`
- `internal/shedadapters#cancelErr`
- `internal/shedadapters#gateAttemptsPointer`
- `internal/shedadapters#gateFailedReason`
- `plan:internal/testkit/shedfake#SeatRunner`

**Intent:** Add the fifth adapter (D1, D5, D7, D8, D11) in `multillm.go`, the thin translation layer over the seat engine.
`SeatRunner` is the handle-shaped seam the recipe's `Env` carries: `Probe`, `Resume` and `Run` over `seatengine.Table`, which `*seatengine.Engine` satisfies, pinned by a compile-time assertion.
`NewMultiLLMProducer` returns a bare pointer like `NewSingleLLMProducer`, defaulting a nil `now` to `time.Now`; the table is validated by the engine on every call and by the recipe entry at construction.
`Call` mirrors `SingleLLMProducer.Call` in order: `entryErr`; probe through the seam before anything is archived; a live chair is resumed and its result mapped; otherwise, with the probe having stopped every live advisor, `archiveStaleOutputs` over `table.Outputs()` with the injected clock, then `Run`, then the map.
A probe, resume or run error returns the context error when the context is cancelled, else the error wrapped with the producer's name and label; an error wrapping `seatengine.ErrSeatNotStopped` is returned without archiving.
`mapOutcome` maps the chair's result exactly as `SingleLLMProducer.mapOutcome` does: Done with the chair's first output as the pointer; a failing gate to Stuck with the pointer, `gateAttemptsPointer` and `gateFailedReason`; died or timeout to an error wrapping `shuttleengine.ErrNotStarted` when `NotStarted` is set; an unrecognized outcome to an error; a genuine success surviving cancellation.
The seat results are logged, never judged: an advisor's death weakens nothing the chair decided.
`multillm_test.go` drives `Call` through `shedfake.SeatRunner`: the probe-before-archive ordering with a live chair leaving the outputs untouched, the fresh path archiving stale outputs of every seat before `Run`, and cancellation at entry and at exit; the chair outcome mapping is one table-driven test whose rows give each shuttle outcome, take the expected mapping from `SingleLLMProducer` driven through `shedfake.Shuttle` over that result, and assert `MultiLLMProducer` over a one-chair table maps it the same, so N = 1 is pinned equal with no separate comparison test.
`doc.go` adds the fifth adapter to its opening and outcome-mapping sections and the probe section, correcting the adapter count where it is spelled.
Raise the `internal/shedadapters` budget row by the tests this card adds.

**ImpactSummary:** One new producer type and seam in the package; SingleLLMProducer, Bouncer, Burler and Webster adapters are untouched, and doc.go's counts change.
