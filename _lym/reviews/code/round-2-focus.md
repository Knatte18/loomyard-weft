---
round: 2
focus:
  - "internal/shedverbs/loopkeeper.go and its tests: does any test fail if the keeper's signal handler is removed, i.e. does a test deliver SIGTERM to the keeper while the loop still runs and assert the exit mark is written and keepLoop returns only after the loop ends?"
  - "internal/shedverbs/step.go DetachedLoopID and the shedcli and loomcli reportArmFailure callers: does a test reach the keeper-flag branch, so reverting the helper to the detached flag alone fails a test?"
  - "internal/shedverbs/loopkeeper.go start-failure path: does a test cover the could-not-start mark, the skipped OnStart and the exit codes 0 and 1 that keepLoop returns?"
  - "internal/shedverbs/loopwait.go deadLoopStop: for a waiter that did not spawn the keeper, can a dead-loop report read the loop log between the loop releasing its lock and the keeper writing the exit mark, and does it then still say loop exit not recorded?"
  - "internal/shedverbs/loop.go LoopSpec field comments for Spawn and OnStart, and doc.go's matching OnStart sentence: do they describe the keeper, the process whose exit closes the Spawn channel and which runs OnStart after starting the loop process?"
---

Focus covers the changed keeper code and its doc twins; it names where to look and restates each site.
