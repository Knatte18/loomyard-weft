# Card 2 — A not-started run carries its start error on `Result`

**Edit:**
- `internal/shuttleengine#Result`
- `internal/shuttleengine#Runner.RunGated`
- `internal/shuttleengine#TestStartup_RunGatedMappings`
- `internal/shuttleengine#TestStartup_RunDeadlineShorterThanWindow_OutputFilePresentAtDeadline`
- `internal/shedadapters#SingleLLMProducer.mapOutcome`
- `internal/shedadapters#TestSingleLLMProducer_NotStartedWrapsErrNotStarted`

**Intent:**
Decision D3.
`RunGated` folds a not-started start into `(result, nil)`, which drops the error `start` returned.
The halted step then shows only "the provider never became ready", while the cause sat in the trace.

- `Result` gains a field `StartError string`, documented beside `NotStarted`: it holds the text of the error `start` returned on `RunGated`'s not-started branch, and it is empty on every other return.
- `RunGated`'s not-started branch sets `result.StartError = err.Error()` next to `result.NotStarted = true`.
  No other site sets it: `Wait`, `finalize` and every other return leave it empty.
- `SingleLLMProducer.mapOutcome`'s `result.NotStarted` branch keeps its `shuttle run outcome <outcome>: %w` text wrapping `shuttleengine.ErrNotStarted`.
  When `result.StartError` is not empty, it appends `: <StartError>` after that text.
  The error still wraps `ErrNotStarted` through the same `%w`, so `errors.Is(err, shuttleengine.ErrNotStarted)` still holds.
  The multi-LLM chair path and burler already carry their start error and are unchanged.

Tests:
- `TestStartup_RunGatedMappings`: both rows ("not-ready start" and "status-cap exit") assert that `result.StartError` contains `ErrNotStarted.Error()` and the strand guid `strand-1`, which `abandonStartup`'s error names.
  The status-cap row also asserts that it contains `errStrandNotTracked`'s text, which only the start error carries.
- `TestStartup_RunDeadlineShorterThanWindow_OutputFilePresentAtDeadline`'s `RunGated_ClassifiesDone` subtest asserts that `result.StartError` is empty on a `RunGated` that started and finished done through `Wait`.
- `TestSingleLLMProducer_NotStartedWrapsErrNotStarted`: the fake shuttle's `Result` carries `StartError: "shuttle: start: the pane said no"` when `notStarted` is true.
  The test asserts that the error's text contains that string when `notStarted` is true, and still asserts `errors.Is(err, shuttleengine.ErrNotStarted)` both ways.

**ImpactSummary:** One additive field on `shuttleengine.Result`, set at a single site in `RunGated`, and one appended clause on the single-LLM producer's not-started error. No reader of `Result` breaks, and the `ErrNotStarted` wrap is unchanged.

**Commit:** 2: A not-started shuttle run carries its start error to the single-LLM producer
