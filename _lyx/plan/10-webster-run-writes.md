# Card 10 — webster-run-writes

**Create:**
- `internal/websterengine#RunWrites`
- `internal/websterengine#loadRunWrites`
- `internal/websterengine/runwrites_test.go`

**Uses:**
- `internal/shuttleengine#WriteEvent`
- `internal/shuttleengine#Engine.AuditForks`
- `internal/websterengine#State`

**Intent:**
One reader of the run's own write history, shared by D5's contract-file evidence and D6's own-path set, in a new `internal/websterengine/runwrites.go`.

`RunWrites` holds `Master []shuttleengine.WriteEvent` (every Master session's own writes) and `Forks []shuttleengine.WriteEvent` (every fork's writes), each event carrying its transcript order's time.
`loadRunWrites` audits every Master session state.json records (`MasterSessionID` and each batch record's `SessionID`, deduplicated, an empty id skipped) with `engine.AuditForks(session, worktree)`, the same facts the audit already parses, and collects `ParentWriteEvents` into `Master` and each fork's `WriteEvents` into `Forks`.
An audit error is returned, naming the session.
It reads only; it records nothing.

Tests, untagged with a `shuttlefake.Engine`: two recorded sessions are both audited and their events merged;
a session id recorded twice is audited once;
an audit error surfaces naming the session.
