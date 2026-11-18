---
round: 1
focus:
  - "Goal item 1, push lock: in internal/fabricengine (bolt.go, pull.go, pushclassified.go, takeoverwrite.go, coalesce.go), does every path that can wait on a pause now pass the gate before taking the push lock? Is any lock-holding path left that waits on a pause hold, including internal/boardcli sync?"
  - "Goal item 1, dobby: 'Dobby starts no batten driver for a paused pair' and its 'Done when' test. The diff touches no dobby package. Does the diff make this change, or rely on existing behaviour with a test that proves it?"
  - "Goal item 2, internal/tenderengine pause.go, resume.go and report.go with tendercli/cli.go: do pause and status both record the pair, report it under skipped and continue when it has no reed session? Is only the no-reed-session error treated this way, and does revive's change stay within the goal?"
  - "Done-when tests: for each of the three, does a test in the diff assert the stated outcome? The lock test needs another pair's batten run to take the lock meanwhile. The pause/status test needs the pair named as skipped."
  - "Test economy and loosened checks: pushdeferral_integration_test.go and coalesce_test.go were rewritten. Do the changed tests still check what their names and doc comments state, and were any existing assertions widened or removed?"
---

Seeded from the decision record's Goal and the diff stat of the review range.
