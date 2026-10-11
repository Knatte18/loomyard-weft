# Card 3 — A not-started run carries its start error to the single-LLM producer

**Edit:**
- `internal/shuttleengine#Result`
- `internal/shuttleengine#Runner.RunGated`
- `internal/shuttleengine#TestStartup_RunGatedMappings`
- `internal/shedadapters#SingleLLMProducer.mapOutcome`
- `internal/shedadapters#TestSingleLLMProducer_NotStartedWrapsErrNotStarted`

**Intent:**
Decision D3.
`RunGated` folds a not-started error into `(result, nil)`, which discards the cause, so a halted step shows only "the provider never became ready" while the trace held the real cause.

`Result` gains a field `StartError string`, documented beside `NotStarted`: the text of the error `start` returned, set only on `RunGated`'s not-started branch and empty on every other return.
`Runner.RunGated` sets `result.StartError = err.Error()` in that branch, beside `NotStarted`.

`SingleLLMProducer.mapOutcome`'s not-started branch appends `: <StartError>` after the existing `shuttle run outcome <outcome>: <ErrNotStarted>` text when `StartError` is non-empty, so `errors.Is(err, shuttleengine.ErrNotStarted)` still holds; an empty `StartError` keeps today's message.
The multi-LLM chair path already returns the seats engine's error with its full text, and burler already carries `StartError` on its halves, so neither changes.

Tests:
- `TestStartup_RunGatedMappings`: both not-started rows assert `result.StartError` is non-empty and contains `ErrNotStarted`'s text and the run dir; add a row whose start reaches ready and whose run then times out in `Wait` on the fake clock (outcome `OutcomeTimeout`, `NotStarted` false), asserting `StartError` is empty on that return.
- `TestSingleLLMProducer_NotStartedWrapsErrNotStarted`: the not-started case sets `StartError` on the fake shuttle's result and asserts the error still wraps `ErrNotStarted` and contains the `StartError` text; the ordinary died case asserts it does not wrap `ErrNotStarted`, as today.

No test asserts the old not-started message text: the repository's only occurrences of `shuttle run outcome` are the producers' own format strings.

**ImpactSummary:** one new `Result` field set on one return, and one producer's not-started message gains the start error's text; every other return and message is unchanged.

**Commit:** 3: Carry a not-started run's start error into the single-LLM failure
