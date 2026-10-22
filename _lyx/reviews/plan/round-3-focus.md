---
round: 3
focus:
  - "Card 5, `Engine.strandsElsewhereLocked`: does a server-wide pane lookup that fails persistently log Warn only on its first failure, Debug while it repeats and Info at recovery, as `PATTERN-log-levels` requires of a polled path (`Engine.Status` and every public op's reconcile are polled)? Does a planned test row reach the repeat?"
  - "Card 7, the worktree skip line in creel's `hubDelivery`: creel's delivery workers and sweep call `Strands` on every probe, possibly concurrently. Does a persistent skip cause log at its Info or Warn level once per path and at Debug on repeats, and is the memory of logged paths safe under concurrency and cleared when the path resolves again? Does a planned test row reach the repeat?"
  - "Card 7, tests: does each planned test function reach a behavior no existing test covers through the public surface, with no test of an unexported helper alone (`PATTERN-test-economy`)? Does every skip cause the card names, including a gitfile naming a missing directory, have a row that reaches it?"
---

The focus follows the round-2 fixes to cards 5 and 7.
