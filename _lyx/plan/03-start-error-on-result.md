# Card 3 — A not-started run carries its start error to the single-LLM producer

**Edit:**
- `internal/shuttleengine#Result`
- `internal/shuttleengine#Runner.RunGated`
- `internal/shuttleengine#TestStartup_RunGatedMappings`
- `internal/shuttleengine/doc.go`
- `internal/shedadapters#SingleLLMProducer.mapOutcome`
- `internal/shedadapters#TestSingleLLMProducer_NotStartedWrapsErrNotStarted`

**Intent:** Decision D3: `RunGated` folds a not-started start into `(result, nil)` and so discards the cause; `Result` now carries it, and the single-LLM producer's failure message shows it.

`shuttleengine.Result` gains a field `StartError string`, documented beside `NotStarted`: the text of the error `start` returned, set only on `RunGated`'s not-started branch and empty on every other return.
`Runner.RunGated` sets `result.StartError = err.Error()` beside `result.NotStarted = true` in that branch; no other return sets it.
With card 2, that text ends with the pane's last lines for a never-ready start.
`doc.go`'s closing startup paragraph adds that RunGated's not-started Result carries the start error's text in `Result.StartError`.

`SingleLLMProducer.mapOutcome`'s not-started return appends the start error after the existing text when `result.StartError` is non-empty: `shedadapters: <name> (<engine>): shuttle run outcome <outcome>: <ErrNotStarted>: <StartError>`, still wrapping `shuttleengine.ErrNotStarted` with `%w`, so `errors.Is(err, shuttleengine.ErrNotStarted)` holds; an empty `StartError` keeps today's message.
The multi-LLM chair path and burler are unchanged: the first already returns the seats engine's full error, and burler already carries its own `StartError`.

Tests:
- `TestStartup_RunGatedMappings`: every row asserts `result.StartError` contains `ErrNotStarted.Error()` when `wantNotStarted` is true and is empty otherwise.
  A new row, `a run that dies after startup has no start error`, scripts `StartupReady` with the status queue live then not-live: the start passes its first probe, `Wait`'s first liveness tick sees the dead pane, and `RunGated` returns `OutcomeDied`, `NotStarted` false, an empty `StartError` and a nil error, with the gate never called.
- `TestSingleLLMProducer_NotStartedWrapsErrNotStarted`: the not-started iteration's `shedfake.Shuttle` result carries `StartError: "shuttle: start: never ready. The pane's last lines:\nboot screen"`, and the test asserts the error still wraps `ErrNotStarted`, contains `shuttle run outcome died: ` followed by `ErrNotStarted.Error()`, and ends with that start error text; the ordinary-died iteration asserts the error contains no `The pane's last lines`.

**ImpactSummary:** One new `Result` field, written at one site; one `shedadapters` error message grows a suffix while keeping its prefix and its `ErrNotStarted` wrap; no caller changes.

**Commit:** 3: A not-started run carries its start error to the single-LLM producer
