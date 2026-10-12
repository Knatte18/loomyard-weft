# Card 15 — batten-arbiter-hold

**Create:**
- `plan:internal/battenshed#ChildArbiter` -> `type ChildArbiter struct`
- `plan:internal/battenshed#TestNotice_LiveArbiterHoldsTheStateChange` -> `func TestNotice_LiveArbiterHoldsTheStateChange(t *testing.T)`

**Edit:**
- `internal/battenshed#InnerRunDeps`
- `internal/battenshed#NewInnerRun`
- `internal/battenshed#innerRunProducer.stateChangeReady`
- `internal/battenshed/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Hold batten's state-changed notice while an arbiter settles the child's stop (D10), so the parent gets no "report none yet" notice for a stop the arbiter may settle.

`ChildArbiter`, in `deps.go`, is the reading of the child's in-flight arbiter marker: `Deadline time.Time` and `Alive bool`, whether the marker's arbiter strand is live.
`InnerRunDeps` gains the field `ArbiterInFlight func(ctx context.Context) (ChildArbiter, bool, error)`, resolved on `Call` like `StopReport`; found false means no marker.
`NewInnerRun` defaults a nil one to a function reporting none.
Batten is told the reading by its caller and reads no config (`PATTERN-told-geometry`).

`stateChangeReady` first reads it: a found marker whose strand is alive and whose deadline plus `noticeReportWait` has not passed returns not ready, for a halted or awaiting child alike.
A read error is a Warn and holds nothing.
A dead strand, a missing marker and a marker past that bound hold nothing, and the existing rules decide as today.
When a `resume` moves the child back to running the episode ends as today, so no notice goes; after a `hand-on` the arbiter verb has removed the marker and the driver's stop report qualifies, so the notice goes.

`TestNotice_LiveArbiterHoldsTheStateChange` drives the notice step with a fake clock over a halted and an awaiting child: a live marker within its bound sends nothing, the same marker past the deadline plus the report wait sends the notice once, a dead strand and a missing marker send it as today, and a marker removed and the child back to running sends nothing.
Raise `internal/battenshed`'s row in the test budget by one.
Describe the hold beside the report wait in `doc.go`.

**ImpactSummary:** one optional dep and one early return in the notice readiness check; a nil dep keeps today's behavior.
