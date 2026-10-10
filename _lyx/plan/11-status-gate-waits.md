# Card 11 — status-gate-waits

**Create:**
- `plan:internal/loomcli#gateWaiting` -> `func gateWaiting(waitDir string, holders func() ([]gateslot.Holder, error), now func() time.Time, next func(st shedengine.Status) (string, error)) func(st shedengine.Status) (string, error)`

**Edit:**
- `internal/loomcli#verifyWaiting` -> `func verifyWaiting(markerPath string, holders func() ([]gateslot.Holder, error), now func() time.Time, next func(st shedengine.Status) (string, error)) func(st shedengine.Status) (string, error)`
- `internal/loomcli#reviewWaitingFor`
- `internal/loomcli/reviewstatus_test.go`
- `contracts/specs/loom-status-spec.md`

**Uses:**
- `plan:internal/hubgeom#GateSlots`
- `plan:internal/gateslot#Pool.Holders`
- `plan:internal/gateslot#ReadWaits`
- `plan:internal/gateslot#WaitDir`
- `internal/verifytree#ReadMarker`

**Intent:**
Make a stall behind another run's gate explainable from the stalled run's own `lyx loom status` (D6).
`verifyWaiting` reads the marker's state: a running verify keeps today's note, and a waiting one renders `<producer>: verify waiting for a gate slot <elapsed> (<site>; holders: <worktree> <site>, …)`, elapsed measured from the wait start, the holders read through the told `holders` function at status time.
`gateWaiting` reports the worktree's live `lyx gate test` wait records the same way, `<producer>: gate test waiting for a gate slot <elapsed> (<command>; holders: …)`, naming the oldest wait, and falls through to `next` when none is live.
`reviewWaitingFor` chains verify, gate test, shuttle, then parent review, with `holders` reading `hubgeom.GateSlots(loc).Holders` and the wait directory `gateslot.WaitDir(loc.AnchorPath())`.
A holders read error renders the note without the holders clause rather than failing status.
Display only: no Go decision reads the waiting state or the wait records (D6's Bound).
`reviewstatus_test.go`'s `TestVerifyWaiting` gains rows for a waiting marker with holders and with a failing holders read, and its chain test gains a gate-test wait row placed between the verify and the shuttle waits.
`loom-status-spec.md`'s `waiting` key section lists the gate-test wait and both waiting forms.

**ImpactSummary:** Two status hooks in one file and their test; the status envelope gains only new `waiting` line forms.
