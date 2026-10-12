# Card 4 — result-start-error

**Edit:**
- `internal/shuttleengine#Result`
- `internal/shuttleengine#Runner.RunGated`
- `internal/shedadapters#SingleLLMProducer.mapOutcome`
- `internal/shuttleengine#TestStartup_RunGatedMappings`
- `internal/shedadapters#TestSingleLLMProducer_NotStartedWrapsErrNotStarted`

**Uses:**
- `internal/shuttleengine#Run.abandonStartup`

**Intent:** D3: a not-started run carries its start error on `Result`, and the single-LLM producer's failure message names it.

`Result` gains `StartError string`, documented beside `NotStarted`: the text of the error `start` returned, set only on `RunGated`'s not-started branch and empty on every other return, Wait's died and timeout outcomes included.
`RunGated`'s not-started branch sets `result.StartError = err.Error()` beside `NotStarted = true`;
`StartGated` and `Run` are unchanged, since `StartGated` returns the error itself and `Run` is `RunGated`.

`SingleLLMProducer.mapOutcome`'s not-started return appends `": " + result.StartError` after the existing `shuttle run outcome <outcome>: %w` text when `StartError` is non-empty, keeping `ErrNotStarted` as the wrapped error so `errors.Is` still holds;
an empty `StartError` leaves the message as it is.
The multi-LLM path and burler already carry the cause and are untouched.

Tests:
`TestStartup_RunGatedMappings`'s two not-started rows assert `StartError` is non-empty and contains the result's run dir, and a third row, "a pane that dies after a ready start is a died result with no start error" (status live then not live, script ready), asserts `OutcomeDied`, `NotStarted` false and `StartError` empty;
`TestSingleLLMProducer_NotStartedWrapsErrNotStarted` becomes a table over `NotStarted` and `StartError`: a not-started result with `StartError` "reed lost the pane" fails with an error wrapping `ErrNotStarted` whose text contains that string;
a not-started result with an empty `StartError` wraps `ErrNotStarted` with the text unchanged;
an ordinary died result wraps nothing and names no start error.

**ImpactSummary:** One exported field, one branch in `RunGated` and one message in the single-LLM producer; `Result` is constructed by value everywhere, so the new field is zero on every existing path and every test fake compiles unchanged.
