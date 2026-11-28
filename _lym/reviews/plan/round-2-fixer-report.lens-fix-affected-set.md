```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that each set-based finding's fix covers every member of its set, re-running the finding's own search.

Findings I left unchecked: F16. Its premise is one code path, not a set.

- **LOW**: card 12, the "Support for the moved tests" bullet ("The rows that read git read through `internal/gitkit`: a bare branch's tip through `gitkit.RevParse`…").
  - Summary: the moved row `ALandingCommitPastOriginHoldsTheRecordsUntilTheRunIsDone` reads bare tips through `fabricengine.BareBranchSHAForTest`, an accessor `boardengine_test` cannot see. The card never names this accessor among the helpers the moved rows replace.
  - Evidence: `internal/fabricengine/pushanchored_integration_test.go:808`, `:818`, `:821` and `:829` call `fabricengine.BareBranchSHAForTest`. It is a `ForTest` accessor that only fabric's own test binary compiles. The generic `gitkit.RevParse` sentence covers it only by implication.
- **LOW**: card 12, the `addPushedPair` sentence ("adds a pair, pushes both sides once and returns its location").
  - Summary: `pairFixture` has the fields `warpPath`, `weftPath`, `warpBranch`, `weftBranch`, `fake` and `location`, and the moved rows use `p.weftBranch` (15 uses) and `p.warpBranch` (5 uses). The card says where the paths come from (hub accessors) but not how a row gets the two branch names when `addPushedPair` returns only a location.
  - Evidence: `internal/fabricengine/pushclasses_integration_test.go:133-152` (`newPairFixtureAt` takes the branches from `hubforge.AddPair`'s `added.Branch`); the moved rows at `pushanchored_integration_test.go:1206-1236` use `p.weftBranch` and `p.warpBranch`.

Checked and covered:
- **F15**: the only package-`fabricengine` test file that reaches `coalescePushSides` is `pushdeferral_integration_test.go`. Card 7 names both of its tests. The other hits (`spawn_test.go`, `spawn_linux_test.go`, `teardownlock_test.go`) stop at the spawn or the pause exit, as card 3 says.
- **F18**: every fabric caller of the deleted `requireOneFailedPush`, `requireNoFailedPush` and `countingBackoff` is in a file on card 12's Edit list (pushclasses, pushanchored, pusherreads, landing, boltpullfirst, rejectedtakeover). `requireTrackingRefPrunedWithUpstreamSet` has no users other than the moved rows.
- **F17**: the only moved row that calls `statusJSON` is `APrimeWeftPushRejectedAfterThisMachinesFinalizeKeepsTheLandingsCommits`, and the card handles it. The moved reconcile rows' `writePairStatus` is replaced by `writeRunStatus`.

Tool calls:
1. Grepped the package-`fabricengine` test files that call the push verbs: F15's set covers tests the review did not list.
2. Read those calls in `spawn`, `spawn_linux` and `teardownlock`, and `DetachedPush`'s pause exit: my context lacked those bodies.
3. Listed the helpers and fixture fields the moved rows' bodies call: my context showed only some of those rows.
