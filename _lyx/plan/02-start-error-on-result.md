# Card 2 — start-error-on-result

**Edit:**
- `internal/shuttleengine#Result`
- `internal/shuttleengine#Runner.RunGated`
- `internal/shedadapters#SingleLLMProducer.mapOutcome`
- `internal/shuttleengine#TestStartup_RunGatedMappings`
- `internal/shedadapters#TestSingleLLMProducer_NotStartedWrapsErrNotStarted`
- `internal/shuttleengine/doc.go`

**Intent:** A never-started run carries its start error to the single-LLM producer's failure message (D3).
`Result` gains a `StartError string`, documented as set only on `RunGated`'s not-started branch to the text of the error `start` returned, and empty on every other return; `NotStarted`'s comment points at it.
`RunGated` sets `result.StartError = err.Error()` beside `result.NotStarted = true` inside its `errors.Is(err, ErrNotStarted)` branch, and nowhere else, so `Run.Wait`, `Attach` and a pre-strand `start` failure leave it empty.
`SingleLLMProducer.mapOutcome`, in the died-or-timeout branch for `result.NotStarted`, appends `: ` and `result.StartError` after the existing `shuttle run outcome <outcome>: <ErrNotStarted>` text when `StartError` is non-empty, still wrapping `shuttleengine.ErrNotStarted` with `%w` so `errors.Is(err, shuttleengine.ErrNotStarted)` holds.
The multi-LLM chair path already returns the seats engine's error with its full text, and burler's own `StartError` is a separate field, so neither is touched.
No new refusal or guard branch is added; the field only carries text the start error already held.
`TestStartup_RunGatedMappings` gains table fields for the expected `StartError` substring and for a pre-strand failure.
Its two existing not-started rows assert `StartError` contains `the provider never became ready` and `result.RunDir`.
A new row sets the fake reed's `AddStrandErr`, so `start` fails before any strand exists and `RunGated` returns that error; it asserts the error is non-nil, does not wrap `ErrNotStarted`, `NotStarted` is false and `StartError` is empty, which pins the empty-elsewhere half of the contract.
The table's existing "error is nil" assertion and its identity assertion (`StrandGUID`, `RunDir`, `SessionID` all populated, which `start` does not fill on an `AddStrand` failure) both become conditional on the new field.
`TestSingleLLMProducer_NotStartedWrapsErrNotStarted` turns its bool loop into rows of `NotStarted` and `StartError`: not-started with a recognizable `StartError` asserts the error contains that text and still wraps `ErrNotStarted`; not-started with an empty `StartError` asserts the error wraps `ErrNotStarted` and does not end in `: `; a run that is not a never-start but carries the same `StartError` asserts the error does not wrap `ErrNotStarted` and holds no start-error text.
`internal/shuttleengine/doc.go`'s startup paragraph, which says a not-ready provider is reported from `RunGated` as `OutcomeDied` or `OutcomeTimeout`, names `Result.StartError` beside it.

**ImpactSummary:** `Result` gains one string field, so every struct literal in the repository stays valid; only `SingleLLMProducer.mapOutcome` reads it, and only on the `NotStarted` branch.

**Commit:** `2: start-error-on-result`
