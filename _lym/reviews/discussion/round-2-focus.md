---
round: 2
exclude_lenses: []
focus:
  - "D6 in _lym/discussion/decision-record.md: the run lock admits the project conversion of the run's own entry. Check that the admitted change is bounded to exactly the derived project of the entry before the write (as endsAsSketch in internal/boardengine bounds a Sketch change, with sketchOf clearing the scope fields), so a write of the converted shape that also changes title, labels or depends_on is refused, and that the record states what the admission lets through."
  - "D6: the skip of status writes after the filing. Check that its key is a run-scoped record removed by battencli.ClearMotifRun, and not the board's raddle/<slug>/ store, which RaddleStore.Holds matches on entry.Sketch and no run clearing removes; settle whether a later beam run under the slug still gets its status writes."
  - "D2 and D4: the split perch's loom keys. Check that any auto-accept key for the perch is stated with its default and guard category, as shedadapters/bouncer.go turns a spent budget into an accept of a CONTINUE or CIRCLING round, and that D4's Bound names what that route lets through and what bounds it."
  - "D4 in perch mode: check that the record says how Raddle-Write ends and stores the cold proposal, and which return handling a re-armed cycle uses, given that loomshed/raddlewrite.go stores the proposal on the gate's pass and moves a returned proposal aside only in auto mode and that raddle-approve is refused in perch mode."
  - "D10: check its rationale against boardengine.Dependencies, which holds a dependency on a plain project until every landing member is done, draft members included; settle which outcomes need dependents re-pointed, and what the run does when the board refuses a re-point."
  - "D7 and Scope Out: check that the refusal's way forward after the filing reaches a working recorder, given filed parts are drafts with no run, and that the refusal's effect on a run whose recipe holds a RaddleFile row but is not the split perch agrees with Scope Out."
  - "D8: check that the member release sits where only lym board set-status reaches it, not in Board.SetStatus, which other null-status writers such as batten cancel's claim clear also call."
---

No lens is excluded: each ran and raised a MEDIUM finding.
