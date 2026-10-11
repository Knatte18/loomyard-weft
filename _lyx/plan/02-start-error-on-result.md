# Card 2 — A not-started run carries its start error on `Result`

**Edit:**
- `internal/shuttleengine#Result`
- `internal/shuttleengine#Runner.RunGated`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine#TestStartup_RunGatedMappings`
- `internal/shedadapters#SingleLLMProducer.mapOutcome`
- `internal/shedadapters#TestSingleLLMProducer_NotStartedWrapsErrNotStarted`

**Intent:** Decision D3.

`Result` gains a field `StartError string`, documented as the text of the error start returned, set only on `RunGated`'s not-started branch and empty on every other return.
`Runner.RunGated`'s `errors.Is(err, ErrNotStarted)` branch sets `result.StartError = err.Error()` beside `result.NotStarted = true`.
No other return sets it.
`internal/shuttleengine/doc.go`'s closing paragraph on the startup probe adds that `RunGated` carries the start error's text on `Result.StartError`.

`SingleLLMProducer.mapOutcome`'s `result.NotStarted` branch appends `result.StartError`, when non-empty, after the existing `shuttle run outcome <outcome>: <ErrNotStarted>` text, as `: <StartError>`, keeping `%w` on `shuttleengine.ErrNotStarted` so `errors.Is` still holds.
The multi-LLM chair path and burler are unchanged.

Tests.
`TestStartup_RunGatedMappings` asserts, on each not-started row, that `result.StartError` contains `ErrNotStarted.Error()` and the strand guid `strand-1`, and on every other row that it is empty.
It gains a row "a start that turns ready and then dies in Wait": status queue live then not-live, startup script `StartupReady`, wanting `OutcomeDied`, `NotStarted` false and an empty `StartError`.
`fastConfig` probes liveness every poll, so Wait's first tick reads the not-live strand and finalizes died.
`TestSingleLLMProducer_NotStartedWrapsErrNotStarted` gives the not-started result a `StartError` text and asserts the returned error contains it while still wrapping `ErrNotStarted`; the ordinary died result carries none and its error does not contain it.

**ImpactSummary:** One additive field on a public struct, one assignment in `RunGated`, and one message suffix in the single-LLM producer; no caller's control flow changes.

**Commit:** 2: Carry a not-started run's start error to the single-LLM failure
