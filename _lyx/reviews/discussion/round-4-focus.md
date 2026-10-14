---
round: 4
focus:
  - "D9's hold end condition in decision-record.md: a Go driver's hand-over appends a settle record of action `hand-over` and then calls `handBack`, which writes the report and the park marker; a successor driver at the same blocked stop routes to the tackler, and a tackler hand-back writes a `hand-back` settle record and a later park marker. Settle whether the decided test for exempting the hand-over's own marker separates it from the successor's marker, and whether the acceptance criteria cover a successor that hands back without an arbiter attempt."
  - "D9's end condition and its acceptance criterion on the signals that end the hold: batten's `StopReport` seam in `internal/battencli/wire.go` reads only the park marker's path and time, while the Go driver's `startReport` writes report files before the tackler and the arbiter run. Settle whether the decided wording names exactly the signal batten reads, and whether the stated reasons for starting the report wait at the hold's end are true."
  - "D9's post-hold report wait: `stateChangeReady` measures the wait from the hold's end. Settle whether every end condition (park marker, hand-on, backstop, applied resume, arbiter switched off) has a named source for that time, and whether a batten restart between the hold's end and the notice preserves it."
---

Focus targets the D9 hold end condition and its timing.
