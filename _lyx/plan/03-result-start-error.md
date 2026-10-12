# Card 3 — A not-started run carries its start error on Result

**Edit:**
- `internal/shuttleengine#Result`
- `internal/shuttleengine#Runner.RunGated`
- `internal/shuttleengine#TestStartup_RunGatedMappings`
- `internal/shuttleengine#TestStartup_RunDeadlineShorterThanWindow_OutputFilePresentAtDeadline`

**Intent:** D3, the shuttle side.
`Result` gains a field `StartError string`, documented beside `NotStarted`: the text of the error `start` returned on `RunGated`'s not-started branch, empty on every other return.
`RunGated` sets `result.StartError = err.Error()` in the `errors.Is(err, ErrNotStarted)` branch, next to `result.NotStarted = true`, and nowhere else; `Start`, `StartGated`, `Wait` and the mechanism-failure return leave it empty.
The field adds no skip or bypass: it only stops `RunGated` from discarding the cause it already folds into `(result, nil)`.
Tests extend existing ones.
`TestStartup_RunGatedMappings` gains a `wantStartErrorHas` per row: both rows (pane-died start, status-cap exit) assert `result.StartError` contains `ErrNotStarted.Error()` and the strand guid, which the real `abandonStartup` text carries; a third row sets `AddStrandErr` on the fake reed, so `start` fails with a non-`ErrNotStarted` error before any strand exists, and asserts `RunGated` returns that error with `result.StartError` empty and `NotStarted` false.
`TestStartup_RunDeadlineShorterThanWindow_OutputFilePresentAtDeadline` already runs `RunGated` to a done outcome in its `RunGated_ClassifiesDone` subtest; it gains an assertion that `result.StartError` is empty, pinning "empty on every other return".

**ImpactSummary:** `Result` is read by `SingleLLMProducer`, the seats engine and burler, which ignore an unread string field; only `RunGated`'s not-started branch writes it.
