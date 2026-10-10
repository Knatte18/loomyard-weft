# Card 1 — logger-debug-at-debug-verbosity

**Create:**
- `plan:internal/logger#durableLevel` -> `type durableLevel struct`
- `plan:internal/logger#durableLevel.Level` -> `func (durableLevel) Level() slog.Level`

**Edit:**
- `internal/logger/logger.go`
- `internal/logger/sink.go`
- `internal/logger/logger_test.go`

**Intent:**
Decision D1: the durable trace sink admits a Debug record when the process verbosity is Debug, set by `-vv` or `LYX_LOG_LEVEL=debug`; Info and above stay admitted unconditionally, and the default stays Info and above.

`durableLevel` is a `slog.Leveler` whose `Level` answers `slog.LevelDebug` while `levelVar` holds Debug and `slog.LevelInfo` otherwise.
`durableHandler.Enabled` compares against `durableLevel{}.Level()` instead of the fixed `slog.LevelInfo`, and `newDurableHandler` builds its inner text handler with `Level: durableLevel{}` instead of the fixed level, so the inner handler follows the same rule.
Nothing else in the fan-out changes: `dualHandler.Enabled` stays the OR of the two halves, and `Handle` keeps its independent re-check.
No config key is added; the enabling switch is the verbosity every lyx process already has.
Bound: a `-vv` process's trace file grows to the existing 8 MiB cap with its single truncation marker; nothing else widens.

Rewrite every statement of the old rule in the package: the package doc's sink paragraph ("every Info+ record also lands", "the first Info-or-above log record"), the `LYX_TRACE=1` bullet, the `LYX_LOG_FILE` duplication note, the Level policy's "The durable sink records Info and above only" line, the `log` var's "gated at Info+ unconditionally" comment, the `durableHandler` type doc, the `Debug` and `Info` function docs ("Debug never reaches the durable sink") and the `SetOutput` doc's "Info+-only" wording.
The new wording: the durable sink records Info and above always, and Debug too when the process verbosity is Debug, so a mutation or step the trace does not record cannot be repaired from, and a `-vv` run leaves its full trace.
`sink.go`'s file comment names the open trigger as "the first Info-or-above record"; it becomes the first record the durable half admits, since at Debug verbosity a Debug record now opens the sink through `durableWriter.Write`.
Comments outside the package that say an Info-or-above record opens the sink (`internal/cliwire/standalone.go`, `internal/fabricengine`) stay true and are not touched.

Tests: in `TestLogging_RoutesRecordsByLevelAndVerbosity` the row "debug at -vv" flips `wantDurable` to true, and the rows "debug at default" and "debug at -v" keep false, so the table pins the whole rule; its doc comment follows.
No new test function, so the package's budget row stays.

**ImpactSummary:** Only the durable half's level gate and the package's own doc statements change; every caller of Debug, Info and Warn is untouched, and only a process at Debug verbosity sees more lines in its trace file.
