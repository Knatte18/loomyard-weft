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
`gateWaiting` reports each of the worktree's live wait records, written by `lyx gate test`, the card-verify rerun and the plan gate's type load (D6): `<producer>: <site> waiting for a gate slot <elapsed>`, one clause per record, oldest first and joined with `; `, then the holders once, `(holders: …)`.
With no live record it returns `next`'s note; with one or more it appends `next`'s note, when non-empty, after `; `, so a live verify marker never hides a gate wait.
The one verify marker per worktree carries every `verifytree.Verify` wait that can occur in a loom run: its Go-side verify sites (Webster's gate, the round gate, Publish, Finalize) are rows of one run and step one at a time, while every other waiter writes its own wait record.
An operator's `lyx webster verify` run beside a loom run's verify shares that one marker and can hide the other's wait, which is today's running-marker behavior and outside this guarantee.
`reviewWaitingFor` chains gate wait, verify, shuttle, then parent review, with `holders` reading `hubgeom.GateSlots(loc).Holders` and the wait directory `gateslot.WaitDir(loc.AnchorPath())`.
A holders read error renders the note without the holders clause rather than failing status.
Display only: no Go decision reads the waiting state or the wait records (D6's Bound).
`reviewstatus_test.go`'s `TestVerifyWaiting` gains rows for a waiting marker with holders and with a failing holders read, and its chain test gains rows for two live wait records rendering both sites, and for a wait record beside a live verify marker rendering the gate wait followed by the verify note.
`loom-status-spec.md`'s `waiting` key section lists the gate wait, naming the sites that write it, and both waiting forms.

**ImpactSummary:** Two status hooks in one file and their test; the status envelope gains only new `waiting` line forms.
