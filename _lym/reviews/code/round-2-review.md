---
verdict: APPROVED
---

No findings.

## Rejected

- Pause-handling coverage (focus 1): `PausedTimeDoesNotCountTowardTheStartGrace` in `TestRunNotice_Conditions` has a dead driver, `Coverage{Intervals: [start, start+2·grace]}` with `Covered` false, and the clock advanced 2·grace, expecting no notice. Dropping `pause.Intervals` from the `AwakeSince` call in `noticeCondition` leaves two minutes of awake time, which is past the grace, so the row would then send a notice and fail. The sleep row (`AMachineSleepDoesNotCountTowardTheStartGrace`) pins the `sleeps` argument the same way.
- Doc paragraphs (focus 2): `internal/dobbyengine/doc.go` and `internal/battenshed/doc.go` both name the loom bootstrap lock as a live-driver signal and the minute of awake time counted from the running episode's since. The takeover bound states the grace counts from first sight. This matches `runNotice`, where a takeover has no memory, so since is taken from the child's status modification time as read on that machine. It also matches `LoomBootstrapLock` being held across spawn and handshake in `internal/loomcli/start.go`.
- Summary claim (focus 3): "Design decisions" says the grace applies whenever dobby sees the child enter running, and that a halt and resume between two ticks keeps the old since. `runNotice` resets `memory.Since` only when `memory.ChildState != child.State` or since is zero, so the claim holds.
- Scope: the diff makes the two smaller fixes the entry names (bootstrap lock counted alive, grace from `memory.Since`) and declines the cleaner reorder; the summary says so. A test covers the running, no-lock child and asserts no notice, as the entry asks.
- Test economy: the bootstrap-lock row in `TestChildDriverLockHeld` and the three grace rows in `TestRunNotice_Conditions` each cover a behavior no other test covers.
