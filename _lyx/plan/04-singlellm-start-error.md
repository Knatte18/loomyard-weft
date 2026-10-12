# Card 4 — Single-LLM producer failure shows the start error

**Edit:**
- `internal/shedadapters#SingleLLMProducer.mapOutcome`
- `internal/shedadapters#TestSingleLLMProducer_NotStartedWrapsErrNotStarted`

**Uses:**
- `internal/shuttleengine#Result`

**Intent:** D3, the producer side.
In the `result.NotStarted` branch of the died/timeout case, `mapOutcome` appends `result.StartError`, when non-empty, after the existing `shuttle run outcome <outcome>: <ErrNotStarted>` text as `: <StartError>`; the error still wraps `shuttleengine.ErrNotStarted`, so `errors.Is` holds, and an empty `StartError` leaves today's message byte for byte.
The multi-LLM chair path already returns the seats engine's error with its full text through `errorExit`, so `MultiLLMProducer.mapOutcome` is untouched.
`TestSingleLLMProducer_NotStartedWrapsErrNotStarted` loops over `notStarted` true and false and is turned into a table of `{notStarted, startError}` cases; the true case builds its `shedfake.Shuttle` `Result` with a `StartError` of `"shuttle: start: not ready — pane showed: boot screen"` and asserts the error text contains it as well as wrapping `ErrNotStarted`, and a second iteration shape with `NotStarted` true and an empty `StartError` asserts the message ends with the `ErrNotStarted` text as before.
The false iteration leaves `StartError` empty and asserts the error does not wrap `ErrNotStarted`.
Greps for the old text found no other assertion on this message.

**ImpactSummary:** One error string in one branch of one producer; the wrap chain is unchanged, so every `errors.Is` on `ErrNotStarted` downstream keeps matching.
