# Card 4 — The single-LLM producer's not-started failure names its start error

**Edit:**
- `internal/shedadapters#SingleLLMProducer.mapOutcome`
- `internal/shedadapters#TestSingleLLMProducer_NotStartedWrapsErrNotStarted`
- `internal/shedadapters/doc.go`

**Uses:**
- `internal/shuttleengine#Result`

**Intent:**
Decision D3, the producer's half: a single-LLM producer whose shuttle start never became ready fails with a message that contains the start error's text.

`SingleLLMProducer.mapOutcome`'s not-started failure keeps its text `shedadapters: <name> (<engine>): shuttle run outcome <outcome>: <ErrNotStarted>` and appends `: <StartError>` when `result.StartError` is non-empty.
It still wraps `shuttleengine.ErrNotStarted` with `%w` exactly as today, so `errors.Is(err, shuttleengine.ErrNotStarted)` still holds.
The multi-LLM chair path and burler already carry the start error and are unchanged.
`doc.go`'s `SingleLLMProducer` bullet says a not-started failure carries the start error's text.

Test: `TestSingleLLMProducer_NotStartedWrapsErrNotStarted` gives the not-started result a `StartError` and asserts the error still wraps `ErrNotStarted` and contains that text; the ordinary died result carries no `StartError`, and its error is unchanged.

**ImpactSummary:** One unexported method's not-started error grows a suffix; its callers in the shed segment match it with `errors.Is`, which still holds, so only the halted step's message text changes.
