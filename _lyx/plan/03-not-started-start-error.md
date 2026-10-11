# Card 3 — A not-started run carries its start error on `Result`

**Edit:**
- `internal/shuttleengine#Result`
- `internal/shuttleengine#Runner.RunGated`
- `internal/shuttleengine#TestStartup_RunGatedMappings`
- `internal/shuttleengine#TestStartup_RunDeadlineShorterThanWindow_OutputFilePresentAtDeadline`
- `internal/shuttleengine/doc.go`

**Intent:**
Decision D3, shuttle's half: `RunGated` folds a not-started start into `(result, nil)` and so discards the cause, which is why the halted step shows only "the provider never became ready".

`Result` gains a `StartError string` field beside `NotStarted`, documented as the text of the error `start` returned, set only on `RunGated`'s not-started branch and empty on every other return.
`Runner.RunGated` sets `result.StartError = err.Error()` in the `errors.Is(err, ErrNotStarted)` branch, beside `result.NotStarted = true`.
No other return sets it, so a `Result` from `Wait`, `Attach` or a pre-strand start failure keeps it empty.
`doc.go`'s startup paragraph adds that `RunGated` carries the start error's text on `Result.StartError`.

Tests:
- `TestStartup_RunGatedMappings` asserts, on both its rows, that `result.StartError` is non-empty, contains `ErrNotStarted`'s text and names `result.RunDir`.
- `TestStartup_RunDeadlineShorterThanWindow_OutputFilePresentAtDeadline`, whose `RunGated` returns a done result, asserts `result.StartError` is empty.

**ImpactSummary:** One additive exported field on `shuttleengine.Result`, set at one site in `RunGated`; existing consumers ignore it, and Card 4 makes the single-LLM producer its first reader.
