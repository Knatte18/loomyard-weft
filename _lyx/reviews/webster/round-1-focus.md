---
round: 1
focus:
  - "Cards 3 and 4 in `internal/seatengine`: does the per-advisor gate, per-seat fork authorization and round number match the card, and does a required advisor's unaccepted end stop the step and set `SeatResult.EndedStep` on every path, including a start failure?"
  - "Card 8 `decideRound` in `internal/burlerengine`: does it choose between the reviewer's rules and the fixer's from `EndedStep` alone, never from outcomes, and are the round's outcome, review files, fixer-report and failure rules unchanged?"
  - "Cards 1 and 5 in `internal/shuttleengine/hold.go` and the chair's deferral: is the held-turn notice deferred only while an advisor is mid-turn or its up line is pending, is the maximum deferral enforced, and does a real chair stall still reach the parent?"
  - "Card 8 `LiveRound`, `Resume` and `ErrHalfNotStopped`: do the exported shapes stay comparable and does `ErrHalfNotStopped` still wrap `seatengine.ErrSeatNotStopped` so `BurlerProducer` is unchanged?"
  - "Cards 7, 10, 11 and 12: do the deletions (`fix_start`, `lyx burler await-review`, `FixStart`, burlerengine's `Shuttle` and `RunnerShuttle`) pass their `assert-no-callers` checks against every target, and do docs and `doc.go` land in the same commit?"
  - "Test economy across the diff: does any new test duplicate an existing test's coverage, and does any added behavior lack a test?"
---

Seed focus derived from the plan overview's card index and shared decisions.
