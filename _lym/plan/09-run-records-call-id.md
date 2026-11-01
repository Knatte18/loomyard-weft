# Card 9 — run-records-call-id

**Edit:**
- `internal/gateslot#RunRecord`
- `internal/gateslot#TestWriteRunRecord`
- `internal/gatecli#execution`
- `internal/gatecli#runTest`
- `internal/gatecli#execution.writeRecord`
- `internal/gatecli#runReport`
- `internal/gatecli#reportCmd`
- `internal/gatecli#TestGateReport`
- `internal/gatecli#TestGateTest_Scenario`
- `//internal/gatecli/doc.go`

**Extends:**
- `//internal/gateslot/doc.go`

**Intent:** D5's record changes: every record of one call carries the call's id, and a record written after the call's context was cancelled is marked abandoned and counted apart.

`gateslot.RunRecord` gains `CallID string` (`json:"call_id,omitempty"`) and `Abandoned bool` (`json:"abandoned,omitempty"`), both optional on read (PATTERN-tolerant-records: a record without them decodes with empty values).
`execution` gains the field `callID`, which `runTest` sets once per call to a fresh id (the call's start time in nanoseconds and the pid, joined), and `execution.writeRecord`, the path every record of a call goes through, the build step's and a blocked tier's included, sets `record.CallID` and sets `record.Abandoned` when `e.ctx.Err() != nil`, whatever cancelled the context: a terminating signal today, a lease or a supersede once the runner exists.
`runReport` counts a record carrying `Abandoned` apart, before its build and carried branches: each row gains `abandoned_runs`, `abandoned_wait_seconds` and `abandoned_wall_seconds`, and such a record adds nothing to `tier_runs`, `carried`, `wait_seconds`, `wall_seconds`, `build_runs`, `build_seconds` or the launch totals.
`reportCmd`'s help and the doc's Report section say so; the gateslot doc's run-record sentence names the call id and the abandoned mark.

Tests:
- `TestWriteRunRecord`, tier0: a record with `CallID` and `Abandoned` reads back equal through `ReadRunRecords`, and a record file written without either key reads back with both empty.
- `TestGateReport`, tier0, gains rows: an abandoned tier record and an abandoned build record count only in the abandoned totals of their row while a finished record of the same key keeps its own totals; records without a call id total as before.
- `TestGateTest_Scenario`, tier1: the row "a two-tier call runs both tiers in turn" also asserts both records share one non-empty `CallID` and neither is abandoned, and the row "a catchable signal kills the tier's whole group and frees the slot" asserts the tier's record carries `Abandoned`.

**ImpactSummary:** two optional record fields and the report's split; every reader decodes tolerantly, and only `lym gate report`'s row shape gains keys.
