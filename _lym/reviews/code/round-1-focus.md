---
round: 1
focus:
  - "Root cause: trace in the code what can still write under the test's temp directories after the old cleanup returned (the OOM watch's first strand read, the name-repair loop, anything either one or enterSession spawns), and settle whether the Root-Cause trailer names that defect and whether waiting on `running` covers every such writer."
  - "Wait completeness: check that every goroutine and subprocess started on a watched session's behalf, including ones started inside watchPaneOOMKills or WatchNames, is joined before `running` reaches zero."
  - "Cleanup ordering: for each test that calls stopWatchedSession, settle whether it runs before every temp directory removal and in a workable order against the fixture's other cleanups (tmux server and session teardown), including the subtest in watchdog_integration_test.go."
  - "Other paths with the same race: check whether any other test reaches enterSession or watchPaneOOMKills, directly or through the daemon loop, and can return while those goroutines still write under its temp directory."
  - "Production behaviour: check the daemon's cancel sites and every construction or copy of watchedSession against the new `running` field, including values built without it."
  - "Claims: check the doc comments on watchedSession and stopWatchedSession, the commit message and _lym/landing/summary.md (including its Merge danger door and blast radius) against what the diff does."
---

The diff is small, so the weight of this round is on whether the wait removes the race the board entry describes,
not only the one path the commit message names.
The decision record's added entry rules out a reproduction run, so the cause has to be settled by reading the code.
