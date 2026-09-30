`.lyx/logs` gets one trace file per lyx invocation that logs at INFO+ or exits non-zero (`internal/logger/sink.go`); `internal/logger/retention.go` keeps the newest 50 files and nothing older than 14 days, both hard-coded.
Operator board upserts and fabric verbs fill the 50 slots, pushing a run's interesting traces out.
Options: make both bounds config keys; or arm the sink only on failure or when a trace id is set (loom runs, ly-drive steps).
Observed in run `operator-surface`: the Webster step's own live trace was swept by retention before the step returned, so the driver's `trace_file` pointed at a missing file.
Retention must never delete the trace of a still-running invocation (skip files held by a live pid, or sweep only at process exit).