# Card 3 — A not-started run carries its start error to the single-LLM producer

**Edit:**
- `internal/shuttleengine#Result`
- `internal/shuttleengine#Runner.RunGated`
- `internal/shuttleengine#TestStartup_RunGatedMappings`
- `internal/shuttleengine#TestStartup_RunDeadlineShorterThanWindow_OutputFilePresentAtDeadline`
- `internal/shuttleengine/doc.go`
- `internal/shedadapters#SingleLLMProducer.mapOutcome`
- `internal/shedadapters#TestSingleLLMProducer_NotStartedWrapsErrNotStarted`

**Intent:**
D3.
`shuttleengine.Result` gains a field `StartError string`, documented beside `NotStarted`: the text of the error `start` returned on `RunGated`'s not-started branch, and empty on every other return.
`RunGated` sets `result.StartError = err.Error()` in the same branch that sets `NotStarted`, and nowhere else.
`internal/shuttleengine/doc.go`'s closing paragraph, which says a not-ready provider is reported as `OutcomeDied`/`OutcomeTimeout` from `RunGated`, adds that the start error's text rides on `Result.StartError`.

`SingleLLMProducer.mapOutcome`'s not-started failure appends `: <StartError>` after the existing `shuttle run outcome <outcome>: <ErrNotStarted>` text when `StartError` is non-empty, so `errors.Is(err, shuttleengine.ErrNotStarted)` still holds and the halted step shows the cause and, through card 2, the pane's last lines.
An empty `StartError` leaves the message as today.
The multi-LLM chair path and burler are unchanged: they already carry the start error's text.

Tests.
`TestStartup_RunGatedMappings`' rows, both not-started, assert `result.StartError` contains `ErrNotStarted.Error()`; they fail today because the field is never set.
`TestStartup_RunDeadlineShorterThanWindow_OutputFilePresentAtDeadline`'s `RunGated_ClassifiesDone` subtest, a `RunGated` call that returns through `Wait` with `OutcomeDone`, also asserts `result.StartError` is empty, so it fails if the field is set outside the not-started branch.
`TestSingleLLMProducer_NotStartedWrapsErrNotStarted` sets `StartError` on the not-started result to a fixed text and asserts, for `notStarted` true, that the error contains that text, besides wrapping `ErrNotStarted`.

**ImpactSummary:** One additive field on `Result`, set at one site in `RunGated`; the single-LLM producer's not-started message grows a suffix, and no other caller of `Result` or of that message is touched.

**Commit:** 3: A not-started run carries its start error to the single-LLM producer
