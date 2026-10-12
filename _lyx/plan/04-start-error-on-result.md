# Card 4 — start-error-on-result

**Edit:**
- `internal/shuttleengine#Result`
- `internal/shuttleengine#Runner.RunGated`
- `internal/shedadapters#SingleLLMProducer.mapOutcome`
- `internal/shuttleengine#TestStartup_RunGatedMappings`
- `internal/shuttleengine#TestStartup_RunDeadlineShorterThanWindow_OutputFilePresentAtDeadline`
- `internal/shedadapters#TestSingleLLMProducer_NotStartedWrapsErrNotStarted`

**Intent:** A not-started run carries its start error to the single-LLM producer's failure message (decision D3).
`Result` gains a field `StartError string`, documented as set only on `RunGated`'s not-started branch to the text of the error `start` returned, and empty on every other return.
`RunGated` sets `result.StartError = err.Error()` beside `result.NotStarted = true`; no other site sets it.
`SingleLLMProducer.mapOutcome` appends `"; start error: " + result.StartError` to its not-started message after the existing `shuttle run outcome <outcome>: <ErrNotStarted>` text when `StartError` is non-empty, so `errors.Is(err, shuttleengine.ErrNotStarted)` still holds; an empty `StartError` leaves the message as it is today.
With card 3 the appended text already ends with the pane's last lines.
The multi-LLM chair path is not touched: it returns the seats engine's error whole, and burler carries its own `StartError`.
The change adds text to a failure message and skips or lets through nothing.
Callers of `mapOutcome` and constructors of `Result` need no edit: the field is additive.
Rows, in existing tests: `TestStartup_RunGatedMappings` asserts `result.StartError` contains `ErrNotStarted`'s text and, on its status-cap row, the status failure's text, on its not-started rows; `TestStartup_RunDeadlineShorterThanWindow_OutputFilePresentAtDeadline` asserts `result.StartError` is empty on its `RunGated` return that goes on to `OutcomeDone`, the return through `Wait`; `TestSingleLLMProducer_NotStartedWrapsErrNotStarted` gives its `shedfake.Shuttle` result a `StartError` when `notStarted` is true and asserts the error message contains it and still wraps `ErrNotStarted`, and for `notStarted` false asserts the message has no `start error` clause.

**ImpactSummary:** One additive field and two assignments; the single-LLM not-started message grows by one clause and no test asserts its old exact text.
