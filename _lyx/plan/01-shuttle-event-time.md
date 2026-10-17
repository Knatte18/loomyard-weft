# Card 1 — shuttle-event-time

**Edit:**
- `internal/shuttleengine#Event`
- `internal/shuttleengine#Runner.ReadEvents`
- `internal/shuttleengine#TestRunner_ReadEvents`
- `internal/shuttleengine#sessionFakeEngine.ParseSessionSignals`
- `internal/testkit/shuttlefake#Engine`
- `internal/shuttleengine/doc.go`

**Create:**
- `plan:internal/testkit/shuttlefake#Engine.ParseSessionSignals` -> `func (e *Engine) ParseSessionSignals(data []byte) ([]shuttleengine.SessionSignal, int)`

**Intent:**
`shuttleengine.Event` gains `At time.Time`, the hook-side time of the line the event was parsed from, zero when the engine gives none (decision record D5, Scope "A shuttle change").
`Runner.ReadEvents` sets it, and is the only place that does: when the run's engine implements `SessionSignalParser`, it calls `ParseSessionSignals` on the bytes read, parses events with `ParseEvents` from `data[:consumed]` only, advances the returned offset by `consumed` instead of to the last newline, and pairs in file order each event with the signals whose `Event` flag is set, giving the event that signal's `At`.
A stamp read in one tick whose Stop payload arrives in the next therefore stays unconsumed and is read with its payload on the next call.
When the marked signals and the events do not pair one to one, every event of that read gets the zero time, so no Stop is stamped from another line.
An engine with no signal reading keeps today's offset rule and gives every event the zero time.
The claude `ParseEvents` and every other `ParseEvents` caller are unchanged.

`internal/testkit/shuttlefake`'s `Engine` gains a `ParseSessionSignalsFn` field and the method `ParseSessionSignals`: with the field nil it reads a line `@<RFC 3339 time>` as the stamp of the next line, gives each line `ParseEvents` would yield as a Stop a signal of kind turn end with `Event` set and the preceding stamp's time (zero with no stamp or a malformed one), and stops `consumed` before a trailing run of stamp lines with no payload after them, so a cycle test over a runner on this kit can confirm a step by its stamp (decision record Scope); as a test kit it carries no test of its own.
The in-package `sessionFakeEngine.ParseSessionSignals` gains the same `@<time>` stamp reading and sets `Event` on its STOP, WAIT and ASK lines, so `TestRunner_ReadEvents` can run over it.

`TestRunner_ReadEvents` gains rows over `sessionFakeEngine`: a stamped Stop gets its stamp's time; a Stop with no stamp gets the zero time; a stamp at the end of the data with no payload is left unconsumed and the next read, after the payload is appended, returns the Stop with that stamp's time; a read whose marked signals outnumber the events gives every event the zero time.
The existing rows over `fakeEngine`, which has no signal reading, pin that such an engine keeps the last-newline offset and gives the zero time.
`internal/shuttleengine/doc.go` states the field and the pairing rule.

**ImpactSummary:** One new field on a shared event type and one reader that sets it; ReadEvents' only production consumer is the orch watcher, whose seam passes the field through unchanged.
