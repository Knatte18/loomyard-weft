# Card 8 — A proven dead Claude process ends the run

**Create:**
- `internal/shuttleengine#Run.sessionGone`

**Edit:**
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.checkLivenessTick`
- `internal/shuttleengine#OutcomeDied`
- `internal/shuttleengine#OutcomeTimeout`
- `internal/shuttleengine/wait_test.go`
- `internal/shuttleengine/startup_test.go`
- `internal/shuttleengine/fakes_test.go`
- `internal/shuttleengine/completionsignal_enforcement_test.go`
- `internal/shuttleengine/doc.go`
- `docs/overview.md`

**Intent:**
D7, with the parent's amendment that a process-ending `SessionEnd` counts only past the first-tick events-file size.

`Run` gains per-wait liveness state: the events-file size taken at the wait's first liveness tick, whether it was taken, and the session ids a liveness tick of this wait read alive.
`Run.Wait` and `Run.awaitStartup` reset it at entry, so each wait starts afresh.

`sessionGone` answers false for an engine without both `SessionSignalParser` and `SessionProber`.
Otherwise, on its first call of a wait it takes the events-file size before anything else, and then:
it parses the events file, takes the session id of the newest signal that names one (else the record's `SessionID`), and reads `ProcessLiveness` for it once.
An unreadable events file counts no `SessionEnd` and uses the record's `SessionID`.
An alive reading records the id as read alive and answers false.
It answers true when either holds:
- the signals parsed from the bytes past the first-tick size end in a process-ending `SessionEnd`, so that line is the newest signal and lies past the first-tick size, and the liveness reading is not alive;
- the reading is dead and this wait read the same id alive at an earlier tick.

Every other case answers false: an unproven reading, a dead reading for an id this wait never read alive, and a `SessionEnd` at or before the first-tick size, whatever the registry later reads.
`Run.checkLivenessTick` calls it on every tick once the strand reads live, before the started short-circuit, so it runs during startup too;
a gone session returns `OutcomeDone` when `allOutputFilesExist`, else `OutcomeDied`, the same path a dead pane takes, with no transient marking, and logs the session id and which proof held at Info.
`awaitStartup` then fails the start as died through `abandonStartup` as it does for a dead pane.
The fold's settle precedence is unchanged.
Bound: an unproven liveness never ends a run; a `SessionEnd` beside an alive registry reading does not; an earlier launch never fails a resumed launch, since its `SessionEnd` lies before this wait's first-tick size and this wait never read its process alive; a wait attached after the death ends only on registry death of an id it read alive, or its deadline.
This adds one events read and one registry read per liveness tick, on the existing cadence (`PATTERN-daemon-wakeups`).

`PATTERN-completion-signal`: the new died return sits in `checkLivenessTick` in `wait.go` behind its `allOutputFilesExist` check, and `completionsignal_enforcement_test.go`'s audited return sites and file-contract call sites gain it.
`OutcomeDied` and `OutcomeTimeout`'s docs, `shuttleengine/doc.go` and the shuttle row of `docs/overview.md` stop saying a provider crash behind a live pane shell classifies timeout.

`TestRun_Wait_Classification` gains the Death criterion rows over a fake engine that parses `SESSIONEND:<reason>` and scripts liveness per tick:
a process-ending `SessionEnd` past the first-tick size with the registry not alive ends the run died, `/exit`'s reason included;
a session read alive whose entry then reads dead ends it died;
with every output file present either ends it done;
an unproven liveness, and a `SessionEnd` beside an alive reading, end nothing;
a crashed launch's dead entry before a resumed launch reads alive ends nothing;
an earlier launch's process-ending `SessionEnd` as the newest signal, with the resumed process not yet registered, ends nothing, and after the wait has read the resumed process alive an unproven reading beside that line ends nothing too.
`startup_test.go` gains a row: a process the wait read alive after its `SessionStart` line that dies before the pane is ready fails the start as died.

**ImpactSummary:** a crashed or exited Claude behind a live pane shell now ends its run died (or done with every output present) on the next liveness tick instead of at the deadline; `Run.Wait`, `Run.awaitStartup` and every attached run share the rule, and engines without the two capabilities are unchanged.
