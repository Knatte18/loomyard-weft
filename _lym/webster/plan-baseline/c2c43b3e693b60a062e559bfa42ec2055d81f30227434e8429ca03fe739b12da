# Card 7 — Shuttle's idle probe reports whether a turn is in progress

**Create:**
- `plan:internal/shuttleengine#guardedReadinessEngine` -> `type guardedReadinessEngine struct`
- `plan:internal/shuttleengine#guardedReadinessEngine.RunningTurn` -> `func (e *guardedReadinessEngine) RunningTurn(capture string) bool`
- `plan:internal/shuttleengine#guardedReadinessEngine.InterruptSettle` -> `func (e *guardedReadinessEngine) InterruptSettle() time.Duration`

**Edit:**
- `internal/shuttleengine#IdleProbe`
- `internal/shuttleengine#readiness`
- `internal/shuttleengine#judgeReadiness`
- `internal/shuttleengine#Runner.SessionIdle`
- `internal/shuttleengine#TestRunner_SessionIdle_ReturnsScriptedClassification`
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine#readinessEngine`
- `internal/shuttleengine#InterruptGuard.RunningTurn`
- `internal/shuttleengine#readBox`

**Intent:**
D4 rings a strand's waiter only when no turn is in progress, whatever its input box holds: a draft does not hold the wake, which `IdleProbe.Idle` cannot say, since a draft makes it false.
D4 also holds that reading false from a ring until a turn end recorded after it, so the probe must say when the turn end it rests on was recorded.

`IdleProbe` gains `NoTurn bool`, true when no turn is in progress whatever the input box holds, and `TurnEndedAt time.Time`, when the hook-derived reading that decided `NoTurn` began, zero when the pane probe decided or a turn is in progress.
`readiness` gains `noTurn bool` and `turnEndedAt time.Time`.
`judgeReadiness` sets them for every state that passes its first switch, the idle, awaiting-input and background-only states, when no turn start is unmatched: `noTurn` true and `turnEndedAt` the state's `Since`, before it reads the box, so a too-short pane, a missing box or a draft keeps `noTurn` true while the reading stays held.
A running turn, a question awaiting an answer, a dead process, an unknown state and an unmatched turn start keep `noTurn` false.
`Runner.SessionIdle` copies them onto its probe for a ready or held reading, the probe its operator-activity hold returns for a ready reading included, so a strand whose operator was recently active reads no turn while its probe stays not idle and is still rung (D4 holds the ring only on a running turn).
When the reading is unknown and the pane decides, `NoTurn` is the pane's idle answer, or, for an engine with `InterruptGuard`, true when `readBox` found the input box, the pane is not too short and `RunningTurn` reads no turn in the capture; a too-short pane or a missing box therefore reads a turn as possible, and `TurnEndedAt` stays zero.

`TestRunner_SessionIdle_ReturnsScriptedClassification` compares whole probes, so each of its rows' wanted probes gains the two fields: idle and draft-held rows from a turn end want `NoTurn` true and the turn end's stamp time, running-turn, ask, dead and unmatched rows want them zero, the operator-activity row wants `NoTurn` true and the turn end's stamp time beside its hold reason, and the unknown-reading and no-run-record rows, whose `readinessEngine` implements no `InterruptGuard`, want `NoTurn` equal to the pane's idle answer: true for the idle pane, false for a pane showing no box and for the busy-draft row.
The existing "a draft in the box is held with its reason" row, updated to want `NoTurn` true with `Idle` false, pins that a draft does not read as a turn.
New rows pin the other boxes under the hook reading: a session idle at a turn end whose pane is too short, and one whose pane shows no input box, each read `NoTurn` true with the turn end's time while `Idle` stays false; the too-short pane under the pane probe is a guarded row, below.
`guardedReadinessEngine`, a test engine in `fakes_test.go`, is `readinessEngine` plus `InterruptGuard`, its `RunningTurn` reading `RUNNING` in the capture, so a row has both the input box and the running-turn check that Claude's pane probe uses.
The test gains a `guarded` engine setting beside `styled`, which runs the row on `guardedReadinessEngine`, and its doc comment names the guarded engine beside the box reader; rows with it set pin the guarded pane probe under an unknown reading: a pane with an input box holding a draft and no running turn reads `NoTurn` true with `Idle` false and a zero `TurnEndedAt`, and the same pane showing a running turn reads `NoTurn` false;
a too-short pane (`tooShort` set) and a pane showing no input box, neither capture showing a running turn, each read `NoTurn` false with `Idle` false and a zero `TurnEndedAt`, so a guarded branch that drops its found-box or not-too-short condition fails a row.
The doc's sentence on what `SessionIdle` answers gains the two fields.

**ImpactSummary:** two probe fields filled inside SessionIdle; its other callers (creelcli's delivery and orch's watcher) read only the old fields and are unaffected until card 8 reads the new ones.
