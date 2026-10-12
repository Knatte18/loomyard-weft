# Card 2 — result-carries-start-error

**Edit:**
- `internal/shuttleengine#Result`
- `internal/shuttleengine#Runner.RunGated`
- `internal/shedadapters#SingleLLMProducer.mapOutcome`
- `internal/shuttleengine#TestStartup_RunGatedMappings`
- `internal/shuttleengine#TestStartup_RunDeadlineShorterThanWindow_OutputFilePresentAtDeadline`
- `internal/shedadapters#TestSingleLLMProducer_NotStartedWrapsErrNotStarted`
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine#Runner.start`

**Intent:** Decision D3 of the decision record: a not-started run carries its start error on `Result`, and the single-LLM producer's failure message shows it.

`Result` gains a `StartError string` field, documented beside `NotStarted`: the text of the error `start` returned, set only on `RunGated`'s not-started branch and empty on every other return, including a died or timeout outcome reached in `Wait` and a pre-strand failure returned as an error.
`RunGated` sets `result.StartError = err.Error()` where it sets `result.NotStarted = true`, and nowhere else; the branch still returns `(result, nil)`.

`SingleLLMProducer.mapOutcome` appends the start error to its not-started failure: the message keeps its existing text, `shedadapters: <name> (shuttle): shuttle run outcome <outcome>: <ErrNotStarted>`, and when `result.StartError` is non-empty ends with `: <StartError>`, so `errors.Is(err, shuttleengine.ErrNotStarted)` still holds.
An empty `StartError` leaves the message byte-identical to today's.
The multi-LLM chair path and burler already carry the start error's text and are untouched.

`internal/shuttleengine/doc.go`'s paragraph on the startup probe, which says a not-ready provider is reported as `OutcomeDied`/`OutcomeTimeout` from `RunGated`, adds that `Result.StartError` then carries the start error's text.

Tests: `TestStartup_RunGatedMappings` asserts on both rows that `result.StartError` is non-empty and contains `ErrNotStarted.Error()`, and on the status-cap row also the text of `errStrandNotTracked`, so the cause reaches the field;
`TestStartup_RunDeadlineShorterThanWindow_OutputFilePresentAtDeadline`'s `RunGated` branch, a done run returned from `Wait`, asserts `result.StartError` is empty, pinning the acceptance criterion that the field is empty on every return other than the not-started branch.
`TestSingleLLMProducer_NotStartedWrapsErrNotStarted` sets `StartError` on the fake's not-started result and asserts the returned error's text contains it, and that the ordinary died result's text is unchanged.

**ImpactSummary:** One new field on Result, set at one site in RunGated and read at one site in mapOutcome; no existing caller of Result or of the Shuttle seam changes shape.
