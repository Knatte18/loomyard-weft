# Card 2 — multillm-table-source-and-preparation

**Edit:**
- `internal/shedadapters#MultiLLMProducer`
- `internal/shedadapters#NewMultiLLMProducer`
- `internal/shedadapters#MultiLLMProducer.Call`
- `internal/shedadapters/multillm_test.go`
- `internal/shedadapters/doc.go`

**Create:**
- `plan:internal/shedadapters#TableSource` -> `type TableSource func() (seatengine.Table, error)`
- `plan:internal/shedadapters#NewMultiLLMProducerSourced` -> `func NewMultiLLMProducerSourced(name string, tables TableSource, seats SeatRunner, now func() time.Time, prepareFreshSpawn func() error) *MultiLLMProducer`

**Intent:** Give `MultiLLMProducer` the two seams D3 names: a per-call table source and a fresh-spawn preparation that returns only an error.
`TableSource` mirrors `SpecSource`: it builds the table for one call, so stencils and directives are read at call time (PATTERN-stencil-ownership).
The change is additive, as the Constraints require while `burler-multi-llm` edits this package: the struct keeps its `table` field and gains `tables TableSource` and `prepareFreshSpawn func() error` beside it, and `NewMultiLLMProducer` keeps its signature and body, setting `table` with no source and no preparation, so every existing caller is unchanged.
`NewMultiLLMProducerSourced` stores the source and the preparation.
`Call` takes its table from the source when one is set, evaluating it once per call before the probe and wrapping a source error as `build table`, and from `table` otherwise; the order on the fresh path is probe, then the preparation when non-nil, then the archive, then the run, as `SingleLLMProducer.Call` orders it, and the preparation never runs on the resume path.
A preparation error returns through `errorExit` as `prepare fresh spawn`, so nothing is archived and no seat starts; the preparation carries no prompt amendment and changes no seat's prompt.
Bound: the preparation is the only hook before the archive and runs only after the probe found no live chair and stopped every live advisor.
Subtests join `TestMultiLLMProducer_ProbesBeforeArchiving`: the source is evaluated exactly once per call and its table is what the probe and the run receive; the preparation runs after the probe and before the archive on the fresh path, observed by stale outputs still present when it runs; the preparation never runs when the probe finds a live chair.
Rows join `TestMultiLLMProducer_ErrorExits`: a source error wraps the cause, runs nothing and archives nothing; a preparation error wraps the cause, archives nothing and runs nothing.
The package doc's `MultiLLMProducer` paragraphs name the sourced constructor, the per-call source and the preparation's place in the call order.

**ImpactSummary:** Additive constructor, type and two struct fields; the existing field, constructor and every current caller keep their behaviour, and only this package's own tests change.
