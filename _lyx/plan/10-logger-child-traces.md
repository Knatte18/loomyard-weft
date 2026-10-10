# Card 10 — logger-child-traces

**Create:**
- `plan:internal/logger#NewTraceID` -> `func NewTraceID() string`
- `plan:internal/logger#TraceFilesFor` -> `func TraceFilesFor(dir, traceID string) ([]string, error)`

**Edit:**
- `internal/logger/trace_test.go`
- `internal/logger/sink_test.go`

**Uses:**
- `internal/logger#mintTraceID`
- `internal/logger#traceIDPattern`
- `internal/logger#traceFilePattern`

**Intent:** The loop of card 11 mints a fresh trace id for each child and hands it through `LYX_TRACE_ID`, so each child keeps its own trace group, and it reads that group's files through a closure the arming module tells it (card 15); this card gives both their `logger` half.
`NewTraceID` is the mint, exported, returning `mintTraceID()` without touching the process's own identity or the environment.
`TraceFilesFor` lists the sink files of one trace group in `dir`: every name `traceFilePattern` matches whose id equals `traceID`, as absolute paths sorted by name, so the oldest file of the group comes first; a group may span several files because a sink file is named by its timestamp and pid, which no caller can spell from the id alone.
An absent `dir` or a group with no file returns an empty list and no error; any other read error is returned.
`trace_test.go`'s adoption test gains a row asserting `NewTraceID` returns a value matching `traceIDPattern`, that two calls differ, and that `TraceID()` is unchanged afterwards.
`sink_test.go`'s `TestEnsureDurableSink_CreatesOneNamedFileWithHeader` gains assertions that `TraceFilesFor` over the sink's directory and the adopted id returns exactly the created file, and returns none for another id and for an absent directory.

**ImpactSummary:** Two new exported functions; the process trace identity, its export and the sink's naming are untouched.
