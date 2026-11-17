---
round: 2
focus:
  - "internal/battencli/leftoverpairs.go `MarkKeptCancels`: when the commit of a written kept cancel intent is refused (merge active) or fails, does the intent stay uncommitted and unpushed because the next call skips a record whose Cancel is set? Settle whether a failed commit is retried."
  - "internal/dobbyengine/orphan.go `dueOrphans`: is an orphan's first-sight time recorded before its notice posts, so a failed post still starts the grace? D7 asks for a teardown no earlier than the grace after the notice."
  - "internal/pairteardown/leftover.go `RemoveHandedOverPair`: does the removal end the pair's reed session before removing the worktree, in the order PATTERN-pair-remote sets for owned processes, reed session, then removal?"
  - "internal/pairteardown/leftover.go `RemoveOrphanPair`: does it re-check the board, the batten run directory and live processes before archiving and removing, or can a cached due list tear down a pair put to use after the last reading?"
  - "internal/battencli/leftoverpairs.go `DobbyOrphans`: does one `fabricengine.PairLastCommit` error fail the whole listing, where an unresolvable worktree is skipped with a Warn?"
  - "Tests for the leftover teardown: for `tearDownLeftover` dispatch and hub workspace regeneration, `RemoveOrphanPair` and `RemoveHandedOverPair`, `DobbyOrphans`, `MarkKeptCancels`, `DobbyLines` kept rows, `PairPushDeferred`, `PairLastCommit`, `DobbyRuns` new fields, the done-run clear's PairPresent skip and the `Orphans` seam failure path, does each behavior D7 names have a test asserting it, and does any new test duplicate another's coverage?"
  - "Docs: does `internal/pairteardown/doc.go` describe `RemoveHandedOverPair` and `RemoveOrphanPair`, does the kept-pairs bullet in `internal/dobbyengine/doc.go` leave the runs bullet's continuation line attached to it, and does the `lym tender resume` help text in `internal/tendercli/cli.go` keep its line width consistent?"
---

Round 2 follows the fixes to the round 1 findings; the entries name where to look and leave the verdict to the reviewer.
