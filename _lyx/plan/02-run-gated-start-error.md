# Card 2 — run-gated-start-error

**Edit:**
- `internal/shuttleengine#Result`
- `internal/shuttleengine#Runner.RunGated`
- `internal/shuttleengine#TestStartup_RunGatedMappings`
- `internal/shuttleengine/doc.go`
- `internal/shedadapters#SingleLLMProducer.mapOutcome`
- `internal/shedadapters#TestSingleLLMProducer_NotStartedWrapsErrNotStarted`

**Intent:**
A not-ready start's cause reaches the single-LLM producer's failure message (decision D3).
`RunGated` folds the not-started error into `(result, nil)`, which today discards the cause, so a halted step shows only "the provider never became ready" while the trace held the real cause.

`Result` gains a `StartError string` field beside `NotStarted`, with a doc comment saying it holds the text of the error `start` returned and is set only on `RunGated`'s not-started branch, empty on every other return.
`Runner.RunGated` sets `result.StartError = err.Error()` on that branch, beside `result.NotStarted = true`, and nowhere else.

`SingleLLMProducer.mapOutcome`'s `result.NotStarted` return keeps its `shuttle run outcome <outcome>: %w` text wrapping `shuttleengine.ErrNotStarted` and appends `: <StartError>` after it when `StartError` is non-empty, so `errors.Is(err, shuttleengine.ErrNotStarted)` still holds.
The multi-LLM chair path already returns the seats engine's error with its full text, and burler already carries its own `StartError`, so neither changes.

`TestStartup_RunGatedMappings` asserts, on both existing rows, that `result.StartError` is non-empty and contains the run dir and `ErrNotStarted`'s text, and gains a row whose start reaches ready and whose pane then goes not-live in `Wait`: outcome died, `NotStarted` false and `StartError` empty.
`TestSingleLLMProducer_NotStartedWrapsErrNotStarted` sets a `StartError` text on the fake shuttle's result and asserts the error contains it when `NotStarted` is true and does not when it is false.

`internal/shuttleengine/doc.go`'s closing paragraph on the startup probe says RunGated reports a not-ready start as `OutcomeDied`/`OutcomeTimeout` with `NotStarted` set and the start error's text in `StartError`.

**ImpactSummary:** one new `Result` field set on one branch; the single-LLM producer's not-started message grows a suffix, and no existing message text it already had is reworded.

**Commit:** `2: A not-started RunGated result carries its start error`
