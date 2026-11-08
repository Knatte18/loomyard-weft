---
round: 3
focus:
  - "D4's Bound and D2 in decision-record.md: with the auto-accept key off and the budget spent, does `softCapRule` in internal/shedadapters/bouncer.go still accept a round whose open findings carry no design-wrong tag, and does the record state that route, what bounds it, and where the open findings go when D2 sets no carry_over?"
  - "D6 in decision-record.md: when a family filing's board write lands but the store write fails or the process crashes before the filing mark is written, can loom's status hook (SetRunStatus in internal/boardengine/board.go) leave a run status on the converted project, and does the record's write order or retry rule keep the project releasable by D8?"
  - "D6's admission in decision-record.md: does it state where the run lock (internal/boardengine/runlock.go, which sees only before and after) reads the filed parts, and does it state which writers and rows the admission is open to and why that is acceptable?"
  - "D2's re-arm and D4's set-aside in decision-record.md: does a re-armed split perch get the segment's full budget given the Bouncer's generation record and amendment allowance (internal/shedadapters/bouncer.go, amendment.go), and do both apply to every fresh entry into Raddle-Write, including a `lym loom goto --to Raddle-Write` as `freshRaddleEntry` in internal/loomcli/raddlewiring.go judges it?"
---

Round 2 left these questions open; each entry restates its sites so it can be settled without earlier reviews.
