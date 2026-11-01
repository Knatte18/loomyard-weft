---
round: 1
focus:
  - "internal/shuttleengine/activity.go `ReadNewestRunEnd`: which timestamp does it take from an ended run's record, and does it cover every non-running outcome (done, failed, cancelled) and a missing or unreadable `run.json` without error or a bogus zero time?"
  - "internal/dobbyengine/runnotice.go `judgeAgents` no-run fallback: is the idle clock the latest of episode `since`, newest history entry and newest agent end, and does a run still `running` keep its existing path unchanged?"
  - "Seam wiring in internal/battencli/wire.go and dobbynotice.go: is `NewestAgentEnd` scoped to the task worktree of the row being judged, and is a nil or unset seam handled as before?"
  - "Notice wording and `since`/idle values after the change: does a long row with a just-ended run stay quiet, and does a row with a truly stale end still notice with correct text?"
  - "Tests: does each added behavior have coverage, and does any new test duplicate an existing one in activity_test.go or runnotice_test.go?"
  - "Commit message and doc.go edits: do the claims match the diff, and do new doc comments follow the repository's conventions?"
---

Scope is the board entry's Idea: a just-ended session counts as fresh activity in the no-run idle clock.
