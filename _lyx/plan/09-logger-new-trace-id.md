# Card 9 — logger-new-trace-id

**Create:**
- `plan:internal/logger#NewTraceID` -> `func NewTraceID() string`

**Edit:**
- `internal/logger/trace_test.go`

**Uses:**
- `internal/logger#mintTraceID`
- `internal/logger#traceIDPattern`

**Intent:** The loop of card 10 mints a fresh trace id for each child and hands it through `LYX_TRACE_ID`, so each child keeps its own trace group; `NewTraceID` is that mint, exported, returning `mintTraceID()` without touching the process's own identity or the environment.
`trace_test.go`'s adoption test gains a row asserting `NewTraceID` returns a value matching `traceIDPattern`, that two calls differ, and that `TraceID()` is unchanged afterwards.

**ImpactSummary:** One new exported function; the process trace identity and its export are untouched.
