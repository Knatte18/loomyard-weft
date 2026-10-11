# Card 2 — A not-started run carries its start error to the single-LLM producer

**Edit:**
- `internal/shuttleengine#Result`
- `internal/shuttleengine#Runner.RunGated`
- `internal/shuttleengine/startup_test.go`
- `internal/shuttleengine/doc.go`
- `internal/shedadapters#SingleLLMProducer.mapOutcome`
- `internal/shedadapters/singlellm_test.go`

**Intent:**
Decision D3: `RunGated` folds a not-started start into `(result, nil)`, which discards the cause, so a halted single-LLM step shows only "the provider never became ready" while the trace holds the real cause.

`Result` gains `StartError string`, documented beside `NotStarted`: the text of the error start returned on `RunGated`'s not-started branch, and empty on every other return.
`Runner.RunGated` sets `result.StartError = err.Error()` in its `errors.Is(err, ErrNotStarted)` branch, beside `result.NotStarted = true`; no other site sets it.
Its doc comment says the folded result carries the start error's text.

`SingleLLMProducer.mapOutcome`, in its `result.NotStarted` branch, appends `result.StartError` after the existing `shuttle run outcome <outcome>: <ErrNotStarted>` text, separated by `: `, when it is non-empty, and still wraps `shuttleengine.ErrNotStarted` with `%w` so `errors.Is` holds.
A not-started result with an empty `StartError` keeps today's message.
The multi-LLM chair path and burler already carry the start error and are unchanged.

`doc.go`'s closing paragraph on the startup probe says `RunGated`'s not-ready result carries the start error's text in `Result.StartError`.

Tests:
- `TestStartup_RunGatedMappings` asserts, on its two not-started rows, that `result.StartError` contains `ErrNotStarted.Error()`.
  It gains a row "a run that dies after a ready start carries no start error": the startup script reads `StartupReady`, the status queue shows the strand live and then not live, so `Wait` classifies `OutcomeDied` with `NotStarted` false and `StartError` empty; the gate is still never called.
  The table gains a `wantStartError bool` column for this.
- `TestSingleLLMProducer_NotStartedWrapsErrNotStarted` sets `StartError` to a fixed text on the not-started fake result and asserts the returned error contains that text and still wraps `ErrNotStarted`; on the ordinary died result `StartError` stays empty and the error contains no start-error text.

**ImpactSummary:** `shuttleengine.Result` gains a field set on one branch of `RunGated`; the single-LLM producer's not-started failure message grows the start error's text, and its `ErrNotStarted` wrap is unchanged.

**Commit:** 2: A not-started run carries its start error to the single-LLM producer
