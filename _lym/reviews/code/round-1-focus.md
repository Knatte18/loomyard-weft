---
round: 1
focus:
  - "Scope: the answer key asks for an anchored-hub row for each lym verb directory call (note verbdir-calls-untested) and a test row for each stop of the steps not yet converted, the weft-dirty stop and the upstream-anchor stop (note anchor-stops-untested). Settle whether the diff's rows cover every such call and both stops, or leave one uncovered. The diff touches only the replaying pair push anchor-change row and the weft-conflict stop; check the commit message's claim against the board entry."
  - "internal/fabricengine/pull.go: the records-conflict stop moved into weftConflictStop. Settle whether the extracted function builds the same WeftStopError as the removed inline literal (Branch, Paths, WayForward, Err), and whether the production refactor is licensed by the answer key, which asks only for tests."
  - "internal/fabricengine/failedpush_test.go, the WeftConflictStopMergesAtTheRecordsRootAndPushesInTheAnchoredDir row: settle whether the hand-built want string duplicates the production wording so a wording change breaks it with no behavior change, and whether an existing test (the weft-dirty row beside it, or a settleUnmovedWeft pull test) already asserts the same way forward. Check that anchoredTask, anchoredWeft and anchoredDir are in scope and anchored for this row."
  - "internal/fabricengine/pushclasses_integration_test.go, AWeftPushThatMustReplayStopsOnAnAnchorChange: settle whether the anchor-change row now asserts the anchored directory (warpPath/backend) rather than only a verb name, and whether the other rows, which keep hubAnchor \".\", still test what they did. Check that newFailureFixtureAt and newPairFixtureAt leave every other caller's behavior unchanged."
  - "Test economy: TestFailedPushRecord and TestPushFailureClasses are listed as not judged by the redundancy report, so judge the new rows by the rubric's bullets: a duplicate of an existing row, a per-helper test the public surface already covers, or a behavior added with no coverage."
---

Seed focus for round 1.
The diff is small: one production extraction in `pull.go`, one export helper, one new unit row and one parametrized integration row.
The decision record holds only the board entry under `## Goal`, so scope is judged against that entry.
