---
round: 3
focus:
  - "internal/loomcli/prepush.go and internal/fabricengine/failedpush.go: does any loomcli code or test still spell the code-repository side value of BranchOutcome.Repo as a literal instead of one exported fabricengine constant?"
  - "internal/loomcli/prepush.go newPrePushCheck and internal/shedrun/paths.go: is the parent-notice record path formed anywhere outside internal/shedrun, and does a shedrun test pin it beside the other scratch paths?"
  - "internal/loomcli/wiring.go loomCommitStatusDeps and pullRequestDeps.PushTask, and the skip closure in newPrePushCheck: does a test drive the real constructors, not stubs, to show Commit, Spawn, Push and PushTask stop for an entry owned elsewhere and the skip closure passes the right reads?"
  - "internal/loomcli/prepush.go stoppedBranches against internal/fabricengine/originreconcile.go classifyEntries: does a stop caused by a shared branch still name the run's own busy records branch?"
  - "pattern/PATTERN-owner-machine.md: does the bullet's exception list match the guard's skips in prepush.go?"
---

Round 3 checks that the round 2 fixes landed and that new text from those fixes is consistent with the repository's rules.
