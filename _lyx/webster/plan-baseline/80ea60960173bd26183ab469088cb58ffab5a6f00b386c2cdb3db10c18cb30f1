# Card 3 — shedrun-loop-files

**Create:**
- `plan:internal/shedrun#LoopLock` -> `func LoopLock(l *lyxcwd.Location, runID string) string`
- `plan:internal/shedrun#LoopPIDFile` -> `func LoopPIDFile(l *lyxcwd.Location, runID string) string`
- `plan:internal/shedrun#LoopLog` -> `func LoopLog(l *lyxcwd.Location, runID string) string`
- `plan:internal/shedrun#LoopEnvelope` -> `func LoopEnvelope(l *lyxcwd.Location, runID string) string`
- `plan:internal/shedrun#LoopDelivered` -> `func LoopDelivered(l *lyxcwd.Location, runID, loopID string) string`
- `plan:internal/shedrun#LoopJobName` -> `func LoopJobName(l *lyxcwd.Location, runID string) string`
- `plan:internal/shedrun#StepTraceCopy` -> `func StepTraceCopy(l *lyxcwd.Location, runID, traceID string) string`
- `plan:internal/shedrun#StepStderr` -> `func StepStderr(l *lyxcwd.Location, runID, traceID string) string`

**Edit:**
- `internal/shedrun/paths_test.go`
- `internal/shedrun/doc.go`

**Uses:**
- `internal/shedrun#StepsDir`

**Intent:** Declare the loop's files where the run directory is declared, per the record's constraint on `PATTERN-shed-run-directory` and `PATTERN-durable-vs-ephemeral-state`: every one sits under `StepsDir(l, runID)`, the run's ephemeral steps directory.
`LoopLock` is `loop.lock`, the liveness record the loop holds for its whole life; `LoopPIDFile` is `loop.pid`; `LoopLog` is `loop.log`, the detached loop's stdout and stderr; `LoopEnvelope` is `loop-envelope.json`, the file the waiter prints and retires; `LoopDelivered` is `loop-envelope.<loopID>.delivered.json`, one slot per loop id that the loop's retired envelope is renamed into, so every waiter on that loop can still print it and a later loop's delivery never replaces it (card 12).
`LoopJobName` is the Windows job-object name, `lyx-loop-` followed by a hex hash of the steps directory's absolute path, unique across hubs and free of path separators.
`StepTraceCopy` and `StepStderr` name the per-step files the loop writes beside the step record, `<traceID>.trace.log` and `<traceID>.stderr.log`, so retention, which sweeps the trace directory alone, never reaches them.
`paths_test.go` gains rows in the path-constructor table for the new constructors, asserts `LoopLock` differs from `RunLock` and `StatusLock` in the lock-distinctness test, and gains a row that `LoopJobName` carries no separator and differs between two locations, and a row that `LoopDelivered` differs between two loop ids of one run.
`doc.go` lists the loop files among the ephemeral run-directory contents.

**ImpactSummary:** Additive constructors in `internal/shedrun`; no existing path changes and no caller is touched.
