# Card 3 — A not-started run carries its start error on `Result`

**Edit:**
- `internal/shuttleengine#Result`
- `internal/shuttleengine#Runner.RunGated`
- `internal/shedadapters#SingleLLMProducer.mapOutcome`
- `internal/shuttleengine#TestStartup_RunGatedMappings`
- `internal/shedadapters#TestSingleLLMProducer_NotStartedWrapsErrNotStarted`
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine#Run.abandonStartup`
- `internal/shuttleengine#ErrNotStarted`

**Intent:**
D3 of the decision record.
`Result` gains `StartError string`, documented as set only on `RunGated`'s not-started branch to the text of the error `start` returned, and empty on every other return.
`RunGated` sets `result.StartError = err.Error()` beside `result.NotStarted = true` and nowhere else; `Run`, `Wait`, `Attach` and `StartGated` never set it.
`SingleLLMProducer.mapOutcome` appends `": " + result.StartError` to its not-started failure message when `StartError` is non-empty (`RunGated` always sets it, so only a hand-built result omits it), after the existing `shuttle run outcome <outcome>: <ErrNotStarted>` text, still wrapping `ErrNotStarted` with `%w`, so `errors.Is(err, shuttleengine.ErrNotStarted)` holds and the halted step shows the cause and, since card 2, the pane's last lines.
The multi-LLM chair path already returns the seats engine's error with its full text and burler already carries `StartError`, so neither is touched.
`doc.go` gains one sentence on `Result.StartError` where it describes `RunGated`'s not-ready report.
The change adds no verb, flag or guard; it only copies an error text the process already holds into a field and a message.

Tests, extending existing ones:
- `TestStartup_RunGatedMappings` gains a `wantStartError` field.
  The row "not-ready start is a died result with a nil error" asserts `result.StartError` contains the `ErrNotStarted` text and the run directory, and the row "status-cap exit" asserts it contains the foreign-strand sentinel's text.
  A new row where the pane is live and ready on the first probe (`StartupScript` of `StartupReady`) and then reports not live on the next status answer ends in `Wait` with `OutcomeDied`, `NotStarted` false and an empty `StartError`, which pins that `RunGated` does not set the field unconditionally.
  The new row's status queue is live, then not live, and the table's shared assertions (identity populated, gate closure uncalled) hold for it unchanged.
- `TestSingleLLMProducer_NotStartedWrapsErrNotStarted` already loops over `NotStarted` true and false with a scripted `shedfake.Shuttle` result; the result also sets `StartError`, and the loop asserts, when `NotStarted` is true, that the error starts with the existing `shuttle run outcome died: ` text, wraps `ErrNotStarted` and ends with the `StartError` text after it, and, when false, that it contains no `StartError` text.

**ImpactSummary:** `Result` is built field-by-field everywhere and compared with `reflect.DeepEqual` only against `Result{}` for the zero case, so a new string field breaks no caller; `RunGated`'s only not-started consumer outside the package that reads `NotStarted` for a message is `SingleLLMProducer.mapOutcome`.

**Commit:** 3: shuttle: carry the start error on Result into the single-LLM failure message
