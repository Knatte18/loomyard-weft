# Card 3 — Every verified send waits for an idle session

**Create:**
- `internal/shuttleengine#ErrSessionBusy`
- `internal/shuttleengine#sendContext`
- `internal/shuttleengine#sendReadyTimeout`
- `internal/shuttleengine#awaitIdleSession`
- `internal/shuttleengine#unmatchedTurnStart`
- `internal/shuttleengine#turnStartHold`
- `internal/shuttleengine#turnStartHold.holds`
- `internal/shuttleengine#turnStartIdleOverride`
- `internal/shuttleengine#paneTail`
- `internal/shuttleengine#paneTailLines`

**Edit:**
- `internal/shuttleengine#sendVerified` -> `func sendVerified(sc sendContext, text string) error`
- `internal/shuttleengine#Run.Send`
- `internal/shuttleengine#Runner.Send`
- `internal/shuttleengine#Runner.LoadSkills`
- `internal/shuttleengine#Run.loadSkillsThenPrompt`
- `internal/shuttleengine#Run.loadSkillTurn`
- `internal/loomcli#driverSender.SendDriver`
- `internal/loomcli#loomCLI.resumeParkedDriver`
- `internal/loomcli#driverResumeSendAttempts`
- `internal/loomcli#driverResumeSendInterval`
- `internal/shuttleengine/run_test.go`
- `internal/shuttleengine/fakes_test.go`
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine#requireReadyAgentPane`
- `internal/shuttleengine#SessionCycler`
- `internal/shuttleengine#SessionSignalParser`
- `internal/shuttleengine#SessionProber`

**Intent:**
Implement D1 and D5's tail for the busy failure: a verified send types nothing until the session is idle in fact, and a session that stays busy fails the send with `ErrSessionBusy` before any key is typed.
The new helpers live in a new file `internal/shuttleengine/send.go`.

`sendContext` carries what every verified send needs: reed, engine, strand guid, the run's events path, the shuttle `Config`, the `Clock` the send's waits sleep and read time through, and a deadline (zero for none; card 6 sets it inside the wait loop).
`sendVerified` takes it in place of its loose arguments and first calls `awaitIdleSession`; its callers build one: `Run.Send`, `Run.loadSkillTurn` and `Run.loadSkillsThenPrompt` from the run (`run.state.EventsPath`, `run.clock`), `Runner.Send` and `Runner.LoadSkills` from the `RunState` their `FindRun` already returns and `r.clock`.
`Run.Send` and `Runner.Send` drop their own `requireReadyAgentPane` call, since the idle wait now owns the readiness guard; `Runner.LoadSkills` loses its "no idle precondition" sentence.
`Runner.Inject` and `Run.Interrupt` are untouched.

`awaitIdleSession`:

- checks the strand live once (`requireLiveStrand`), then, for an engine without the idle reading (`SessionCycler`), runs today's `requireReadyAgentPane` and returns its answer: no window, no signal read, so its refusal still wraps `ErrPaneNotReady`.
- otherwise polls the pane until it reads idle in fact: `Startup` classifies it `StartupReady` and `IdleSession` passes, and, for an engine that implements `SessionSignalParser`, `unmatchedTurnStart` finds no `turn_start` after the newest turn end in the events file, or `turnStartHold.holds` releases the one it found.
- polls with backoff through `sc.clock`, from 250 ms doubling up to 2 s, until `sendReadyTimeout(sc.cfg)` has passed or `sc.deadline` (when set) has, whichever is first.
- on expiry returns an error wrapping `ErrSessionBusy` that names the reading that failed (pane not ready, pane not idle, or the unmatched `turn_start` with its time) and ends with `paneTail`.

`unmatchedTurnStart` reads the whole events file through the engine's `ParseSessionSignals` and returns the newest `turn_start` with no later `turn_end` or `api_error_turn_end`; an unreadable file or an engine without the parser reads as none.
`turnStartHold` remembers the `turn_start` it is holding and since when the pane has read idle on every poll; `holds` answers false once the engine's `SessionProber.TurnStartInterrupt` reports that turn interrupted, or once the pane has read idle on every poll for `turnStartIdleOverride`, logging one Warn naming the unmatched signal when it releases it that way.
An engine with signals but no `SessionProber` has only the 10 s path.
A not-idle poll resets the idle timer.
`sendReadyTimeout` returns the configured window, flooring a non-positive hand-built value to the template default 60 s as `pollInterval` does.

`paneTail` captures the pane once and returns its last `paneTailLines` non-blank lines, or a sentence saying the final capture failed with its error; it is the D5 tail every send failure ends with.

Bound (this card adds a wait and a new refusal): the wait can only delay a send or fail it as busy; it never ends, classifies or finalizes a run, and reads only the raw `turn_start` signal, never the fold's derived state.
It is capped by `send_ready_timeout_s` and, inside the wait loop, by the run deadline; an unmatched `turn_start` holds a send against an idle pane for at most `turnStartIdleOverride`.

`loomcli`'s `driverSender.SendDriver` and `resumeParkedDriver` doc comments are updated: against Claude a busy or not-ready pane now waits inside the send and fails with `ErrSessionBusy`, which the resume loop does not retry; `ErrPaneNotReady` remains the refusal of an engine without the idle reading, which the loop still retries.
The comment on `driverResumeSendAttempts` and `driverResumeSendInterval` is updated the same way: they bound the wait for an input-ready pane only for an engine without the idle reading, while against Claude the send's own `send_ready_timeout_s` window does.
`doc.go`'s send paragraph describes the idle wait.

Tests in `run_test.go`, through a fake engine in `fakes_test.go` with the idle reading, session signals and `TurnStartInterrupt` scripted, and a `fakeClock` on the run: a new table test of the idle wait, with rows for a running-turn pane that turns idle (typed only after), a draft that never clears (`ErrSessionBusy`, no key typed, error carries the pane's last lines and a failed final capture says so), an unmatched `turn_start` over an idle pane (held, then released at 10 s with one Warn), the same released by `TurnStartInterrupt`, and an `api_error_turn_end` counting as a turn end.
An engine without the idle reading keeping today's refusal gets no row: `TestRun_InterruptAndSend_RefuseAgentlessShellPane` already pins `Run.Send` on such an engine refusing with `ErrPaneNotReady` and keeps passing unchanged.
`Runner.Send` gets one row of the same table showing it waits for idle as `Run.Send` does.
Existing send tests keep their rows; their fakes have no idle reading, so they take the unchanged guard.

**ImpactSummary:** `sendVerified` is re-signed and its five callers in `run.go` and `session.go` build a `sendContext`; every verified send (`Run.Send`, `Runner.Send`, `Runner.LoadSkills`, the skill-load and prompt sends in start) now waits for idle, and three `loomcli` doc comments follow.

**Commit:** 3: shuttle: a verified send waits for an idle session, fails busy with the pane's last lines
