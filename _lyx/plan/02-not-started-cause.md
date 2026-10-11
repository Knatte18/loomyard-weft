# Card 2 — not-started-cause

**Edit:**
- `internal/shuttleengine#Result`
- `internal/shuttleengine#Runner.RunGated`
- `internal/shuttleengine#TestStartup_RunGatedMappings`
- `internal/shuttleengine#TestStartup_RunDeadlineShorterThanWindow_OutputFilePresentAtDeadline`
- `internal/shedadapters#SingleLLMProducer.mapOutcome`
- `internal/shedadapters#TestSingleLLMProducer_NotStartedWrapsErrNotStarted`

**Intent:**
A not-started run carries its start error on `Result` (decision D3).
`RunGated` folds a not-started start into `(result, nil)`, which discards the cause, so a halted step shows only "the provider never became ready" while the trace held the real cause.

`shuttleengine.Result` gains a field `StartError string`, documented beside `NotStarted`: the text of the error start returned on `RunGated`'s not-started branch, empty on every other return.
`Runner.RunGated` sets `result.StartError = err.Error()` on the branch where `errors.Is(err, ErrNotStarted)` holds, beside `result.NotStarted = true`; no other return sets it.

`SingleLLMProducer.mapOutcome`'s not-started branch appends `result.StartError`, when non-empty, after the existing `shuttle run outcome <outcome>: <ErrNotStarted>` text, so `errors.Is(err, shuttleengine.ErrNotStarted)` still holds and the message carries the cause.
The multi-LLM path already returns the seats engine's error in full, and burler already carries its own `StartError`, so neither changes.

Tests:
- `TestStartup_RunGatedMappings` asserts, on each not-started row, that `result.StartError` is non-empty and contains `ErrNotStarted`'s text.
- `TestStartup_RunDeadlineShorterThanWindow_OutputFilePresentAtDeadline`'s `RunGated_ClassifiesDone` subtest asserts `result.StartError` is empty on a run that started and finished.
- `TestSingleLLMProducer_NotStartedWrapsErrNotStarted` gives the not-started case a `Result.StartError` text and asserts the error both wraps `ErrNotStarted` and contains that text; the ordinary died case keeps no `StartError` and asserts the error does not contain it.

**ImpactSummary:** One new `Result` field set at one site in `RunGated` and read by one producer's not-started message; every other `Result` reader sees an empty string.

**Commit:** 2: Not-started runs carry their start error to the single-LLM producer
