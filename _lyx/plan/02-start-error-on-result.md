# Card 2 — A not-started run carries its start error on Result

**Edit:**
- `internal/shuttleengine#Result`
- `internal/shuttleengine#Runner.RunGated`
- `internal/shuttleengine#TestStartup_RunGatedMappings`
- `internal/shuttleengine/doc.go`
- `internal/shedadapters#SingleLLMProducer.mapOutcome`
- `internal/shedadapters#TestSingleLLMProducer_NotStartedWrapsErrNotStarted`

**Intent:** Decision D3: `RunGated` folds a not-started start into `(result, nil)` and so drops its cause; the cause now rides on `Result` to the single-LLM producer's failure message.

`Result` gains a `StartError string` field beside `NotStarted`, with a doc comment: the text of the error `start` returned, set only on `RunGated`'s not-started branch, which `Runner.Run` reaches through `RunGated`, and empty on every other return.
`Runner.RunGated`'s `errors.Is(err, ErrNotStarted)` branch sets `result.StartError = err.Error()` beside `result.NotStarted = true`.
No other site sets it.

`SingleLLMProducer.mapOutcome`'s `result.NotStarted` branch keeps its `shuttle run outcome %s: %w` text wrapping `shuttleengine.ErrNotStarted`, and appends `: ` and `result.StartError` after it when `StartError` is non-empty, so `errors.Is(err, shuttleengine.ErrNotStarted)` still holds.
The multi-LLM chair path and burler are unchanged: they already carry the start error's text.

`internal/shuttleengine/doc.go`'s Start/StartGated/Run/RunGated paragraph says Run's and RunGated's not-ready result carries the start error's text in `Result.StartError`.

Tests:
- `TestStartup_RunGatedMappings` asserts, on its two not-started rows, that `result.StartError` contains `ErrNotStarted`'s own text, and gains a row "a run that dies after starting carries no start error": status queue live then not live on `%1`, startup script `StartupReady`, so `awaitStartup` returns ready and `Wait`'s first liveness tick (`fastConfig` probes every poll) finalizes `OutcomeDied`; the row wants `OutcomeDied`, `NotStarted` false and `StartError` empty, and the gate closure is still never called.
- `TestSingleLLMProducer_NotStartedWrapsErrNotStarted` sets `StartError` to a fixed text on the not-started iteration and asserts the error contains it, and asserts the not-not-started iteration's error does not.

**ImpactSummary:** One new `Result` field set at one site; `mapOutcome` appends it only on its not-started branch, keeping the `ErrNotStarted` wrap every `errors.Is` caller (`shedtransient`, shed's halt) reads; no other `Result` consumer reads the field.

**Commit:** 2: A not-started run carries its start error to the single-LLM producer
