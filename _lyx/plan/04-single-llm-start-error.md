# Card 4 — single-llm-start-error

**Edit:**
- `internal/shedadapters#SingleLLMProducer.mapOutcome`
- `internal/shedadapters#TestSingleLLMProducer_NotStartedWrapsErrNotStarted`

**Uses:**
- `internal/shuttleengine#Result`

**Intent:** The single-LLM producer's not-started failure names its cause (D3).

In `mapOutcome`'s died/timeout branch, the `result.NotStarted` return keeps its existing text, `shedadapters: <name> (<engine>): shuttle run outcome <outcome>: <ErrNotStarted>`, wrapping `shuttleengine.ErrNotStarted` with `%w` as today, and appends `: <result.StartError>` after it when `result.StartError` is non-empty.
`errors.Is(err, shuttleengine.ErrNotStarted)` still holds; the ordinary died/timeout return and the other branches are unchanged.
The multi-LLM chair path and burler already carry their start error, so only this producer changes.

`TestSingleLLMProducer_NotStartedWrapsErrNotStarted` sets `StartError` on the fake shuttle's `Result` for both iterations, a text such as `shuttle: start: the provider never became ready — run dir /r, strand "g"`, and asserts the returned error's text contains it when `NotStarted` is true and does not when it is false, beside its existing `errors.Is` assertion.
The `false` iteration pins that the append lives inside the `NotStarted` branch; `RunGated` itself never sets `StartError` without `NotStarted` (card 3).

**ImpactSummary:** One branch of one method in `internal/shedadapters/singlellm.go`; every single-LLM shed step's not-started failure message gains its cause after the unchanged prefix, so a reader matching the prefix or `ErrNotStarted` still matches.
