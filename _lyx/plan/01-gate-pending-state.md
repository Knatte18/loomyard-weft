# Card 1 — Gate results gain a pending state

**Edit:**
- `internal/shuttleengine#GateResult`
- `internal/shuttleengine#GateEntry`
- `internal/shuttleengine/gate.go`
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine#Run.evaluateGate`
- `internal/shuttleengine#Run.finalize`
- `internal/shuttleengine/doc.go`

**Create:**
- `internal/shuttleengine/gate_pending_test.go`

**Uses:**
- `internal/shuttleengine#Run.Send`
- `internal/shuttleengine/fakes_test.go`

**Intent:**
Add decision D2's third gate-result state to the gate contract and teach the wait loop to hold on it.

`GateResult` gains `Pending bool`, `Send string` (one line for the loop to send the writer; the closure never sends), and `SendFailedWayForward string` (the clause the loop's Warn quotes when that send fails).
`GateEntry` gains `Final Gate`: the closure `finalize` evaluates in place of `Gate` when non-nil, so a pending-capable entry can be evaluated without its opening and consuming transitions.
`GateEntryState` gains `GateEntryWaiting` (`"waiting"`).

`evaluateGate` takes whether it runs for `finalize` and picks `Final` over `Gate` then.
Evaluation stops at the first failed or pending entry; a pending entry is reported `waiting`, sends no re-prompt and changes no count, and later non-off entries are `not_reached` as today.
A pending result from an entry that is not `PassOnCap` is a returned gate error (`shuttle: gate: entry %q returned pending but is not pass_on_cap`), so pending can never let a must-pass gate through.
`waiting` counts toward `Passed` like `let_through`.

`Run` gains the turn-boundary state: whether the writer's last arrival was Done with nothing sent since, the last Done message, and the index of the pending entry.
`Wait` factors the gated-Done handling into one helper reached from two places: a gated Done arrival, and a poll tick with no new arrival while an entry is pending and the writer is at a turn boundary.
The helper evaluates the gate list from the first entry; a pass finalizes with the remembered Done message; a failure with budget re-prompts exactly as today; a pending result sends its `Send` text when non-empty, marks the writer mid-turn on success, and keeps polling.
A failed send of a pending text logs one Warn carrying the entry name, the error and `SendFailedWayForward`, leaves the entry pending and the writer at the boundary, and never ends the loop (unlike a failed re-prompt send).
Every gate send therefore happens only at a turn boundary; a verdict recorded mid-turn is read at that turn's Done.
Liveness and the run deadline keep running through a pending wait; a deadline or liveness finalize evaluates through the `Final` closures and reports the entry `waiting`.
If `completionsignal_enforcement_test.go`'s audited sites shift, update its pinned list in this card.

Bound: pending holds the run only while the deadline Start set from the spec has not expired, never extends it, and only a `PassOnCap` entry may return it.

`gate_pending_test.go` drives `Wait` over the package's fake engine and fake clock: a pending Done keeps polling and sends the carried text once; a failed carried send warns, keeps the entry pending and changes no count; a poll tick mid-turn never re-evaluates or sends; a reject after pending re-prompts once; a later pass finalizes Done with the remembered message; a pending result from a non-`PassOnCap` entry is an error; a deadline during pending finalizes with the entry `waiting` and calls `Final`, never `Gate`.
Update the gate paragraphs in `doc.go`, `gate.go` and `wait.go`'s file comment.

**ImpactSummary:** The gate contract types and the gated branch of the wait loop change; existing ungated and fail/pass gate paths keep their behaviour, and every current `GateEntry` literal compiles unchanged since `Final` is optional.
