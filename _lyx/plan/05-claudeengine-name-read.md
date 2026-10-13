# Card 5 — Session-name read failures: missing transcript at Debug, others once per session

**Create:**
- `plan:internal/shuttleengine/claudeengine#nameReadWarnings` -> `type nameReadWarnings struct`
- `plan:internal/shuttleengine/claudeengine#nameReadWarnings.first` -> `func (s *nameReadWarnings) first(sessionID string) bool`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude`
- `internal/shuttleengine/claudeengine#New`
- `internal/shuttleengine/claudeengine#NewFromConfig`
- `internal/shuttleengine/claudeengine#Claude.SessionNameDrift`
- `internal/shuttleengine/claudeengine#TestSessionNameDrift_MissingTranscriptReportsNoneAndLogs`
- `internal/shuttleengine/claudeengine#TestSessionNameDrift_UnreadableTranscriptReportsNoneAndLogs`

**Intent:**
D10, first line: `claudeengine: could not read the session name`, which the hub watchdog's name repair reaches on every idle tick, logs a transcript that does not exist yet at Debug, and any other read failure at Warn once per session id for as long as the provider value lives.

`nameReadWarnings` is a mutex-guarded set of session ids; `first` records the id and answers true the first time it sees it, and a nil receiver answers true, so a `Claude` built without a constructor warns as it does today.
`Claude` gains a `nameReads *nameReadWarnings` field, a pointer so the struct stays safe to copy and share across concurrent runs, and `New` and `NewFromConfig` set it; `Claude`'s doc comment, which says its methods are pure functions of fixed fields, names this one piece of shared mutable state.
In `SessionNameDrift`, a `forEachTranscriptLine` error that `errors.Is(err, fs.ErrNotExist)` logs at Debug;
every other error, `claudeProjectDirFor`'s included, logs at Warn only when `c.nameReads.first(sessionID)` answers true, and at Debug otherwise.
The method's answer is unchanged: every read failure still reports no drift.
The file's header comment, which says a changed `custom-title` shape makes the watchdog log each retype, is unaffected, since that path reads a transcript successfully.

`TestSessionNameDrift_MissingTranscriptReportsNoneAndLogs` captures at Debug verbosity and asserts the could-not-read line is present at `level=DEBUG` and no `level=WARN` line is logged.
`TestSessionNameDrift_UnreadableTranscriptReportsNoneAndLogs` calls `SessionNameDrift` twice for `sid` on one `New()` value and asserts exactly one `level=WARN` could-not-read line, then once for a second session id, `sid2`, whose transcript a second `writeSessionTranscript` call writes and the test replaces with a directory the same way, on the same value, and asserts a second `level=WARN` line.
It captures at Debug and asserts the repeated read of `sid` logged at `level=DEBUG`.
Neither test is parallel, since both redirect the process-global logger.

**ImpactSummary:** one struct field set by both constructors and one method body in `claudeengine`; callers see the same drift answers, and only the log level and repetition of read failures change.

**Commit:** 5: claudeengine: log a session-name read failure once per session
