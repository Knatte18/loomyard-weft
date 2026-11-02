---
round: 1
focus:
  - "internal/gitexec/export_test.go `SetStallCheckForTest` against the board entry's idea: does the stall row now stay independent of git start-up time on every path, including the retry attempt, whose readiness is `count >= attempt` over a count file shared across attempts?"
  - "internal/gitexec/gitexec_unix_test.go stall row: with `ready` true the check runs `os.Stat(tracePath)` and requires size 0; settle whether the trace file always exists by the time the ext command has counted, and whether a false `ready` can leave the poll goroutine running or the attempt waiting until the 10x deadline."
  - "internal/gitexec/gitexec_unix_test.go 'a remote that answers is not stalled' row: the check now fires only once the trace is non-empty, so settle whether the row still exercises the production check's size-0 condition or passes trivially."
  - "internal/gitexec/gitexec.go `startStallTimer` seam: confirm the default preserves the old `time.AfterFunc(...).Stop` and `defer` behavior exactly, and that a doc comment's claim about `tracePath` being unused holds."
  - "Test economy: the added `armed()` assertions in all three rows; settle whether each checks a behavior the row did not already cover."
  - "Commit message claims against the diff, and whether the 200 ms stall bound in the stall row still proves what the test is meant to prove (a silent remote is killed and retried once)."
---

Single commit; the answer key is the board entry's idea in the decision record, which carries no amendments.
