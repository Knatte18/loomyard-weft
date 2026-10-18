# Card 1 — shuttle-event-time

**Edit:**
- `internal/shuttleengine#Event`
- `internal/shuttleengine#Runner.ReadEvents`
- `internal/shuttleengine#TestRunner_ReadEvents`
- `internal/shuttleengine#sessionFakeEngine`
- `internal/shuttleengine#sessionFakeEngine.ParseSessionSignals`
- `internal/testkit/shuttlefake#TestEngine_InertDefaults`
- `internal/shuttleengine/doc.go`
- `internal/testkit/shuttlefake/shuttlefake.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `internal/testkit/shuttlefake#SignalEngine`
- `internal/testkit/shuttlefake#SignalEngine.ParseSessionSignals`
- `internal/testkit/shuttlefake#TestSignalEngine_PairsStampsAndHoldsTrailingStamps`

**Intent:**
`shuttleengine.Event` gains `At time.Time`, the hook-side time of the line the event was parsed from, zero when the engine gives none (decision record D5, Scope "A shuttle change").
`Runner.ReadEvents` sets it, and is the only place that does: when the run's engine implements `SessionSignalParser`, it calls `ParseSessionSignals` on the bytes read, parses events with `ParseEvents` from `data[:consumed]` only, advances the returned offset by `consumed` instead of to the last newline, and pairs in file order each event with the signals whose `Event` flag is set, giving the event that signal's `At`.
A stamp read in one tick whose Stop payload arrives in the next therefore stays unconsumed and is read with its payload on the next call.
When the marked signals and the events do not pair one to one, every event of that read gets the zero time, so no Stop is stamped from another line.
An engine with no signal reading keeps today's offset rule and gives every event the zero time.
The claude `ParseEvents` and every other `ParseEvents` caller are unchanged.

`internal/testkit/shuttlefake`'s `Engine` stays as it is and implements no `SessionSignalParser`, since shuttleengine branches on the capability itself and not only on the signals it yields: an inert parser would still turn on `send.go`'s landing wait, move `ReadSessionStates`' reading off `SessionCauseUnsupported` and make the session shadow read signals instead of logging them unavailable, for every consumer of the kit.
The fake engine the decision record's Scope asks for is a new type in `shuttlefake.go`, `SignalEngine`, which embeds `*Engine` and adds `ParseSessionSignals`, so only a test that builds one takes the parser branches: it reads a line `@<RFC 3339 time>` as the stamp of the next line, gives each line on which the embedded `ParseEvents`, handed that line alone, yields an `EventStop` a signal of kind turn end with `Event` set and the preceding stamp's time (zero with no stamp or a malformed one), and stops `consumed` before a trailing run of stamp lines with no payload after them, so a cycle test over a runner on this kit can confirm a step by its stamp.
A test that builds a `SignalEngine` sets the embedded `ParseEventsFn` to a per-line reading, since the default `ParseEvents` answers the fixed `Events` whatever its input; the type's doc comment states that requirement.
`TestEngine_InertDefaults` asserts that `Engine` implements no `SessionSignalParser`, and the package doc in `shuttlefake.go` names `SignalEngine` as the kit's opt-in signal-reading engine.
`TestSignalEngine_PairsStampsAndHoldsTrailingStamps`, a new untagged test in `internal/testkit/shuttlefake`, runs over a `SignalEngine` whose `ParseEventsFn` reads one line: a stamped Stop line yields a turn-end signal with `Event` set and the stamp's time, a Stop with no stamp or a malformed one yields the zero time, and `consumed` stops before a trailing run of stamp lines; `cmd/lyx/testdata/test-budget.yaml` raises `internal/testkit/shuttlefake` by one.
The in-package `sessionFakeEngine` gains a `markEvents bool` field, set only by the new `TestRunner_ReadEvents` rows.
With it set, `ParseSessionSignals` reads `@<time>` stamps the same way and sets `Event`, with the stamp's time, only on the lines `fakeEngine` and `waitingEngine` `ParseEvents` yield an event from: `STOP:<message>`, `WAIT:<message>` and `ASK:<message>`, the last read as an ask; a bare `ASK` stays an ask without `Event`.
It also reads a `MARK` line, which no `ParseEvents` yields an event from, as a turn end with `Event` set, so a row can build a read whose marked signals outnumber its events.
With it unset, every reading is today's, so the dialog rows of `dialog_test.go` and every other test over the fake keep their results.

`TestRunner_ReadEvents` gains rows over `sessionFakeEngine` with `markEvents` set: a stamped Stop gets its stamp's time; a Stop with no stamp gets the zero time; a stamp at the end of the data with no payload is left unconsumed and the next read, after the payload is appended, returns the Stop with that stamp's time; a read holding a `MARK` line beside a stamped Stop, whose marked signals outnumber the events, gives every event the zero time.
The existing rows over `fakeEngine`, which has no signal reading, pin that such an engine keeps the last-newline offset and gives the zero time.
`internal/shuttleengine/doc.go` states the field and the pairing rule.

**ImpactSummary:** One new field on a shared event type and one reader that sets it; ReadEvents' only production consumer is the orch watcher, whose seam passes the field through unchanged; the kit's consumers keep their readings because `shuttlefake.Engine` gains no parser, and the in-package fake's stamp reading is opt-in, so no other shuttleengine test changes.
