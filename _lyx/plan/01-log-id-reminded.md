# Card 1 — log-id-reminded

**Edit:**
- `internal/creelengine#Store.Log`
- `internal/creelengine/doc.go`
- `internal/creelcli#logCmd`
- `tools/sandbox/SANDBOX-CREEL-SUITE.md`
- `internal/creelengine_test#TestScheduler_Reminders`

**Uses:**
- `internal/creelengine#LogFilter`
- `internal/creelengine_test#env.start`
- `internal/creelengine_test#env.probeWaits`
- `internal/creelengine_test#env.post`
- `internal/creelengine_test#env.readBodies`
- `internal/creelengine_test#eventsOf`
- `internal/creelengine_test#strand`

**Intent:**
D4: `Store.Log`'s id clause keeps an entry whose `ID` equals `filter.ID` or whose `Unread` holds `filter.ID` (`slices.Contains(entry.Unread, filter.ID)`); a zero `filter.ID` still filters nothing.
The `From`, `To` and `Since` clauses apply to every entry as before, so a `reminded` entry, which carries `To` and no `From`, survives `--to` and is dropped by `--from`.
Since `#664` a `reminded` record carries its ids in `Unread` and no `ID`, so before this change `log --id` never listed one.
Update `Store.Log`'s doc comment to say the id filter also keeps an event whose unread ids hold the id.

D5: in `internal/creelengine/doc.go`'s `# Log` section, add one line: `--id` also lists the `reminded` events whose unread ids hold that id.
In `tools/sandbox/SANDBOX-CREEL-SUITE.md`'s S9 `Watch:` paragraph, the `log --id <id>` expectation `(`posted`, `read`)` becomes the message's events `posted`, each `reminded` event whose unread ids hold it, and `read`, oldest first.
In `logCmd`, the `--id` flag help `show only the events of the message with this id` and the Long text's `--id limits it to one message` both gain the reminders that list the id, for example `show only the events of the message with this id and the reminders that list it`; these are static help strings, so no test asserts them.

Test: add one row to `TestScheduler_Reminders`, `LogByIDListsTheRemindedEventsThatHoldTheID`:
`e.start(t)`; `first := e.post(t, strand(alpha), beta, "one")`; `e.probeWaits(1)` (the reminder listing `first` is journaled once the worker sleeps again);
`e.readBodies(t, strand(beta), "", false)` reads `first`;
`before := len(e.probeWaits(0))`; `second := e.post(t, strand(alpha), beta, "two")`; `e.probeWaits(before + 1)` (a reminder listing only `second`).
Assert, with `eventsOf`:
`Log(LogFilter{ID: first})` is `posted, reminded, read` (the reminder listing `second` is left out);
`Log(LogFilter{ID: second})` is `posted, reminded`;
`Log(LogFilter{ID: second, To: beta})` is `posted, reminded`;
`Log(LogFilter{ID: second, From: alpha})` is `posted`.
These are the full lists: `Reading.Ack` journals only a `read` event, and a worker probes before it first waits, so the reminder listing `first` is journaled once `e.probeWaits(1)` returns.

The existing `LogFilter{ID: ...}` assertions were checked against this change: those in `verbs_test.go` and `server_test.go` run no scheduler or filter an inject id, and `deliver_test.go`'s filter mail whose receiver is an orch strand or a removed strand, so none of them sees a `reminded` event and none changes.

**ImpactSummary:** one filter clause in `Store.Log`, reached by the daemon's log verb only, plus `logCmd`'s help wording; no record shape changes and no existing assertion changes.

**Commit:** 1: creel log --id lists the reminded events that hold the id
