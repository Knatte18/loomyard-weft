# Card 3 — result-start-error

**Edit:**
- `internal/shuttleengine#Result`
- `internal/shuttleengine#Runner.RunGated`
- `internal/shuttleengine#TestStartup_RunGatedMappings`
- `internal/shuttleengine#TestStartup_RunDeadlineShorterThanWindow_OutputFilePresentAtDeadline`

**Intent:** A not-started run carries its start error on `Result` (D3).

`Result` gains a field `StartError string` after `NotStarted`, with a comment: the text of the error `start` returned, set only on `RunGated`'s not-ready branch, and empty on every other return.
`RunGated`'s `errors.Is(err, ErrNotStarted)` branch sets `result.StartError = err.Error()` beside `result.NotStarted = true` and still returns `(result, nil)`; no other return sets it.
`Result` is built by `finalize`, `identity` and the zero literal elsewhere, and none of those set the field, so it is empty there by construction.

Tests:
`TestStartup_RunGatedMappings` gains a `wantStartError` substring per row and asserts `result.StartError` contains it: `ErrNotStarted.Error()` for the not-ready row, and `did not track strand` for the status-cap row, whose start error names reed's mechanism failure.
`TestStartup_RunDeadlineShorterThanWindow_OutputFilePresentAtDeadline`, whose `RunGated` returns a done run from `Wait`, asserts `result.StartError == ""`.

**ImpactSummary:** One new exported field on `shuttleengine.Result`, written at one site in `RunGated`; existing readers of `Result` (shedadapters, burler, seats, the CLI) are unaffected, and card 4 is its first reader.
