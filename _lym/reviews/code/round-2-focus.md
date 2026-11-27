---
round: 2
focus:
  - "Landing routes in the code: read landingshed's beam pull-request route, the darn route and every other path by which work reaches main or a parent branch, and settle for each whether the tests held back by `//lym:only parent-align finalize` have run on the tree that lands. Judge from the code, not from the PATTERN-landing rule's statement."
  - "Marked-scope upkeep key (`internal/gatecli/only.go` `markedScope` and `withOnlyTests`, `internal/gatecli/run.go` `runTest`, `internal/gatecli/tiers.go` `planTierRuns` and `scopeInvocations`): under each combination of listed tiers, `--no-carry`, a tier's `carry: false`, explicit targets, filter flags after `--`, scoper value and a missing go.mod or marked test, does the call run exactly its marked tests, and does a call that runs fewer say so by its exit code?"
  - "Held tests and the green record (`withoutHeld`, `recordPass`, `recordInertCarried`, `greenRecord.with`): can a pass that held tests back be carried by a key that holds back less, and does a test fail if the guard step stops dropping a held guard or an entry stops recording `held`?"
  - "Board-tidy upkeep call (`internal/beamcli/dobbydeps.go` `UpkeepCall`, `internal/dobbyengine` `spawnTidy` and its tidy stencil): is the call rendered only where it can run, and does the stencil tell the agent what to file when the call is refused or runs nothing, not only when a test fails?"
  - "Whole-repository scans: sweep the test files, `cmd/lym` first, for any test walking the whole repository that carries neither a place marker nor a watch marker, and settle whether each still runs at task gates against what the decision record asks."
---
