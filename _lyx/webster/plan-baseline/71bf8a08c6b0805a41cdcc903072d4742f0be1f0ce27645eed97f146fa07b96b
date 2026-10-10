# Card 7 — seatengine-probe-resume

**Create:**
- `internal/seatengine#LiveTable`
- `internal/seatengine#Engine.Probe`
- `internal/seatengine#Engine.Resume`
- `internal/seatengine#Engine.probeSeat`
- `internal/seatengine#resumeNotice`
- `internal/seatengine/resume_test.go`

**Edit:**
- `internal/seatengine/doc.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/seatengine#Engine.join`
- `internal/seatengine#stopSeat`
- `internal/seatengine#noticeSender`
- `internal/burlerengine#Engine.ProbeRound`
- `internal/burlerengine#Engine.Resume`
- `internal/shuttleengine#Runner.ProbeGated`

**Intent:** Probe a table's seats before anything is archived, and resume a live chair (D8), in `resume.go`.
`LiveTable` carries the chair's handle, nil when the chair is not live, and the live advisors' handles keyed by seat name.
`Engine.Probe` validates the table and probes every seat through `probeSeat`, the chair with the table's gate and each advisor ungated, both through the superseded-removing `ProbeGated` shape, so an advisor whose record is terminal and whose strand is still up, a Done advisor kept by `KeepPane` or a timed-out one, is stopped by shuttle's own removal, keeping its record's outcome, and answers not found.
When the chair is live `Probe` returns its handle with every advisor the probe found; when it is not, `Probe` stops every live advisor through `stopSeat` and returns an empty `LiveTable`, so a caller archives and starts fresh only after no seat is live, and a failed stop is `ErrSeatNotStopped` as in card 6.
`Engine.Resume` requires a live chair, starts the notice sender on the chair's handle, joins the found advisors as card 6 does, and queues one `resumeNotice` line for every advisor the probe did not find: an advisor whose outputs all exist is finished and its line says the output stands and the seat no longer answers, any other is ended and its line says so as a mid-step death does; an advisor not found is never restarted.
A notice sent before the interruption may land again, which costs the chair nothing.
`resume_test.go` covers the probe-and-resume acceptance list with card 6's fake: a live chair attached and running advisors joined, an ended advisor not restarted and the chair told once with the right wording for a finished and for an ended advisor, a live chair with a kept Done advisor whose probe stops it and leaves its outputs in place, a chair not live stopping the live advisors and archiving nothing itself, nothing stopped while the chair is live except what the probe removes, and a strand holding a seat name after the probe reported as a start error naming the holder.
Extend `doc.go` with the probe-before-archive ordering and the resume rules.
Raise the `internal/seatengine` budget row by the tests this card adds.

**ImpactSummary:** doc.go grows; the new verbs reuse card 6's join, sender and stop without changing them.
