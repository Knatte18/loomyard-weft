---
round: 1
focus:
  - "internal/sheddriver/drive.go and driverules.go: with the watched verify line removed, does a watched hand-back still write the stop report, record the hand-back settle record and park the run, and does the unwatched verify halt keep its one goto and run-id hand-back line?"
  - "internal/sheddriver/drivehandback.go: after the field removals, is any code, comment or doc comment left that still describes a parent message for the watched verify halt, and is every driver notice path for a watched run (relayed parent notice, question, friction-failed done) still reachable?"
  - "internal/loomcli/doc.go and the other docs naming the Go driver's messages to a watched run: do they agree with the new behavior and with the change description's claims?"
  - "The three changed tests (rule table, notice set, drive loop): does each pin that no notice is sent without leaving a now-dead fixture field or an assertion that duplicates a sibling test's coverage?"
---

Seed focus from the board entry (drop or reword the parent message for a watched verify halt) and the diff stat across `internal/sheddriver` and `internal/loomcli/doc.go`.
