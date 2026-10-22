---
round: 5
focus:
  - "D7 in `_lyx/discussion/decision-record.md`: for each of its three sites (batten owner probe, hibernate wake, `lyx orch start`), settle what restarts the Go driver chain that `End` ended once the run reads owned here again, and whether the Decision and its acceptance criterion name that mechanism and pin it. Check the named command against `internal/loomcli/resume.go`, which spawns a driver only for a child whose status is not running, and against `lyx adopt`'s `battencli.ResumeInWindow` and `internal/battenshed/doc.go`."
  - "D6 in `_lyx/discussion/decision-record.md`: settle when a marked push child reads its coverage exemption (the parent lock record, read through `HoldsParentLock` in `internal/tendercli/gate.go` over live `parentlock.Store.HeldBy` records) against when batten releases that record (`internal/battenshed/innerrun_parentlock.go`), and what happens to a push spawned by a Finalize past its parent-side merge that checks after the record is gone, including what respawns it, given the lift spawn excludes a pair another machine owns."
---

Focus targets the two keys still open after round 4.
