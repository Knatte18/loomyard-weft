---
round: 2
focus:
  - "internal/reportengine/batten.go `snapshotsOfRun` and `widen`: does a run in the window with no earlier tag and no fresh seed still widen the records walk to the whole history, and does that lower the board read's floor in `readLandings` (landings.go) so board.json is read at every commit?"
  - "internal/reportengine/boardhistory.go `readBoardHistory` and batten.go `battenIndex.walk`: does each `gitrepo.Log` call stop at the floor, or still traverse every commit of the branch?"
  - "internal/reportengine landing joins (`readLandings`, `landingJoiner.timeJoinOwners`, `boardHistory.labelsAt`): do the join results, namely a checkpoint older than the records walk's floor opening a time-join window and labels held only by an older board commit, match what the unbounded reads produced, and does the result depend on whether another run widened the walk?"
  - "internal/reportengine/boardhistory.go line 52: is the board in effect at the floor taken from committer-time order, and can skewed committer times across machines select an older entry's labels than ancestry would?"
  - "internal/reportengine tests `TestLandingJoiner_HoldsNoParsedBoard`, `TestCompute_ReadsBattenStatusesOnlyOverEachRunsSpan` and `TestCompute_ReadsEachBoardCommitOnceFromTheWindowOn`: does each carry the `//lym:seam` and `//testtiming:keep` markers PATTERN-test-speed requires, and does each pin the Since of the records walks, the floor-lowering loop in `readLandings`, the tip as board in effect, and the widening stop at an earlier tag?"
  - "internal/reportengine/doc.go package doc: does its description of where the first records walk opens match `Compute`'s `landingWalkStart` and `ComputeTiming`'s `Since`?"
---

Focus restates the open design findings of round 1 as questions to settle against the code.
