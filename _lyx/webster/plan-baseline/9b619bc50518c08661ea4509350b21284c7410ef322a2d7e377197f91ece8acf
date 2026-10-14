# Card 10 — multillm-amendment

**Edit:**
- `internal/shedadapters#MultiLLMProducer`
- `internal/shedadapters#NewMultiLLMProducerSourced` -> `func NewMultiLLMProducerSourced(name string, tables TableSource, seats SeatRunner, now func() time.Time, prepareFreshSpawn func() (string, error)) *MultiLLMProducer`
- `internal/shedadapters#MultiLLMProducer.Call`
- `internal/shedadapters#MultiLLMProducer.mapOutcome`
- `internal/shedadapters/doc.go`
- `internal/shedadapters#TestMultiLLMProducer_ProbesBeforeArchiving`
- `internal/shedadapters#TestMultiLLMProducer_ErrorExits`
- `internal/shedadapters#TestMultiLLMProducer_MapsTheChairResultAsSingleLLMDoes`
- `internal/shedrecipe#discussionSeatsEntry`

**Intent:** D10: `MultiLLMProducer` gains what `SingleLLMProducer` alone has, so every writer row can move onto it.

- `NewMultiLLMProducerSourced`'s fresh-spawn preparation returns a prompt amendment beside its error, as `SingleLLMProducer`'s does.
  `Call` runs it on the fresh path only, after the probe found no live lead and before the archive, as today, and sets the returned text as the table's `Amendment` (card 9), so the seat engine appends it to the lead's prompt; the resume path neither runs the preparation nor amends.
  The table the source returned is copied before the field is set, so a source that hands out one table value is never mutated.
- `mapOutcome`'s not-started error carries the lead's `StartError` text after `shuttleengine.ErrNotStarted` when shuttle reported one, as `SingleLLMProducer` does, so the operator reads why the provider never came up.
- `MultiLLMProducer`'s field and `doc.go`'s MultiLLM paragraph describe both; the producer still maps the lead's outcome exactly as shuttle reports it and judges nothing itself (PATTERN-completion-signal).

`discussionSeatsEntry`, the one production caller, returns an empty amendment from its parent-review preparation.

Tests:
- `TestMultiLLMProducer_ProbesBeforeArchiving`: its preparation closures return `("", err)`; a new row returns a non-empty amendment and asserts the table the runner's `Run` received carries it as `Amendment` while the table its `Probe` received carries none, and the resume row asserts the preparation never ran and the resumed table carries no amendment.
- `TestMultiLLMProducer_ErrorExits`: its failing preparation returns `("", boom)`.
- `TestMultiLLMProducer_MapsTheChairResultAsSingleLLMDoes`'s existing never-came-up row gains a `StartError` on the lead result and asserts the error wraps `shuttleengine.ErrNotStarted` and its text holds the start error.

**ImpactSummary:** One re-signed constructor with one production caller in shedrecipe; Call and mapOutcome change on the fresh and not-started paths only.
