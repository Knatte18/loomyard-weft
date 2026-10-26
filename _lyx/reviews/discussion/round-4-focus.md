---
round: 4
focus:
  - "D9's InnerRun bullet in decision-record.md: for a child parked at Beam-Await, settle what notice a driver found dead produces, and what bounds the respawns of a driver that re-enters the row, writes its status file and dies again, given that `noticeCondition` in `internal/battenshed/notice.go` reports a driver-dead finding for every running child whose driver is dead and that `internal/battenshed/innerrun.go` resets the respawn count when a status write falls after the last respawn. Check the `lyx beam create` acceptance criteria against the answer."
  - "D9's failed-accept path in decision-record.md: it restarts the beam's dobby when the dobby-stop marker stands, then removes the marker and the request and posts the failure. Settle what happens when that restart returns an error, what the beam's orch is told, and whether an acceptance criterion covers the case."
  - "D5's last Upgrade sentence in decision-record.md: it says an older build's writes to a beam project entry carrying recipe pawl fail. In `internal/boardengine`, `normalizeTask` runs `checkProjectFields`; check which write paths reach it and which do not, and whether the sentence states that correctly."
---

Each entry names the decision to re-read and the question to settle.
The findings behind them were raised in round 3; round 4 reviews the text as the fixer leaves it.
