---
round: 1
focus:
  - "internal/sheddriver/drive.go and drivehandback.go: does `judged` reach every hand-back of a blocked stop the tackler or arbiter judged, including a stop that moves to another stop after the judgment, and is it reset at the right point in `goDriver.run`? The survivor at drive.go:250 (`d.judged = false` dropped) needs a check for a test that would kill it."
  - "internal/sheddriver/drivehandback.go `driveNotices`: with `judgedBlocked` and a watched run, can the parent now hear two notices for one stop (batten's own plus the run-id line), and does the doc comment match the code?"
  - "internal/loomshed/gates.go `overviewFrameEditedFindings`: does the frame comparison use the same hash and the same trigger conditions (begun batch, recorded frame) as Webster's rebaseline refusal, so the gate and the rebaseline agree in both directions? Check an unreadable overview and a record with no frame."
  - "internal/webstercli/reset.go, wiring.go, sync.go: does the reset's records commit carry `Run-Transition` for every reset path (to batch-start, to start with archive only, standalone), and is `shedrun.ResolveRunID` the right run id source in hub mode? The decision record also names `lym loom goto --to Webster` writing no transition; check whether the diff licenses leaving it out."
  - "internal/fabricengine/commit.go `CommitWithTrailers` and `commitWeftLocked`: validation before staging, trailer order after Warp-SHA, behaviour on an unborn warp and on a no-op weft commit; reach report says no mapped test reaches the new functions, so check what TestCommitWithTrailers and the webstercli reset test actually assert."
  - "Test economy: `TestDriveNotices` is listed as contained, covered by `TestGoDriver_Settle`. Compare their assertions before filing a fold-or-keep finding."
  - "Commit trailers: each `Root-Cause:` names the defect the diff removes; check the three trailers against their diffs."
---

Seed focus drawn from the decision record's three defects and the review range's four commits.
