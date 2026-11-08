---
round: 4
focus:
  - "Decision-record D2 and D4: does the record's rule for a goto into Raddle-Write match the entry `shedengine.Goto` writes (internal/shedengine/goto.go, Producer set to the target row, Outcome goto, RoutedTo empty), and does `freshRaddleEntry` in internal/loomcli/raddlewiring.go recognise that entry, so the last proposal is moved aside?"
  - "Decision-record D2: when the Raddle-Write perch re-arms on a goto, does the record state what happens to the spent bounce count and the recorded verdict (internal/shedengine/run.go goto handling; Bouncer.Call in internal/shedadapters/bouncer.go) for a perch that had not settled, with a stated bound?"
  - "Decision-record D4 and the D2 acceptance criteria: does the soft-cap bound describe exactly what `designWrongKeys` in internal/shedadapters/bouncerfiles.go counts as open design-wrong (open, BLOCKING or MEDIUM, not tagged mechanical), and what the rule therefore files unreviewed?"
  - "Decision-record D6: does the bound on the lock's open admission (internal/boardengine/runlock.go checkRunLocks, which sees only before and after) name the writers and rows it admits and state what Raddle-File does when the converted parts differ from the run's proposal (internal/loomshed/raddlefile.go Call and AlreadyFiled)?"
  - "Decision-record D6 retry rule: does it cover a crash after the store write and before the filing mark, where Raddle-File's Call finds the store done and `finish` could end the converted project done?"
---

The focus names the open gating questions only.
