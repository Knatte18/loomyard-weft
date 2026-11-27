---
verdict: APPROVED
---

The diff is confined to `internal/gatecli/testmap_integration_test.go`.
It makes alpha's hanging fixture hang only on the `testMapHangingEnv` cue, and it widens the map calls' per-test limit from 3 s to 10 s through the named `testMapSecondUnit`.
Every lens and the handler's own review came back with no kept finding.

## Focus settlement

- **Paths to `passed_alone`, `timed_out` or `failed_tests` under load.**
  In every call but the `--all` call that sets the cue, alpha's tier1 pair are both healthy, so no test waits out a limit.
  `passed_alone` arises in `runBatch` (`internal/gatecli/testmappackage.go:296-308`) only when a test of a batch of two or more, run side by side, fails or times out and then passes alone.
  The hanging test plays no part in that mechanism.
  What exposed `TestAlphaTwo` was the 3 s limit, which was kept short so that every call's hang stayed cheap.
  With the hang confined to one call, the limit is now 10 s in every call.
  The `Root-Cause` trailer names that coupling, and the diff removes it.
  Every call keeps a residual margin, as any test under a production per-test limit does; see Rejected.
  A non-timeout failure of a healthy stand-in has no new path: the HangsOnCue case without the cue falls through to the normal profile write and `--- PASS`.
- **Subtests that read alpha's map or compare exit codes.**
  `AnAllCall…` still pins the timed-out outcome, the failed-test entry and the map's `OutcomeTimedOut` entry.
  `NamedPackageIsRebuiltWhenFresh` still checks the rebuild through `BuiltAt`.
  `AMapRunBuildsNoLym` still checks that no lym is built, and that every run saw an empty prebuilt path.
  `AnEllipsis…` keeps exit 1 through outcomes' failing test, broken's build and zmoved.
  `WhileATestHangs` sets the cue, so the test still hangs.
  No later subtest relied on the timed-out map entry: those that read alpha's map compare `BuiltAt` alone.
- **What `testMapSecondUnit` governs.**
  In a map call it reaches only `testTimeout` (`internal/gatecli/testmap.go:338`).
  No build, mutant or call bound in these tests derives from it.
  The 2 s context of `AMapRunHoldsTheLargerSlotAskOfItsTiers` ends while the call waits for a slot, before any test runs.
  No map call in this file still uses 150 ms.
  The sibling reach and redundancy steps keep their own 150 ms unit; the amended entry does not name them.
- **Trailers and summary.**
  `Root-Cause` names the defect the diff removes.
  `Loosened-Check` names the widened limit and cites the verify log's recorded outcome, not "flaky".
  The summary's claims hold against the code: every other call waits out no limit, the cue call waits twice, `WhileATestHangs` already set the cue, and the `Hangs` users (`TestYieldingHangs`, `TestDropsHangs`) are unchanged.

## Rejected

- code-scope's MEDIUM finding that `AnAllCall…` still rests on a margin against the parent's "deterministic" decision: the residual exposure is not specific to that call.
  Any batch of two or more side-by-side tests under any per-test limit has it: outcomes' four tier0 tests in every `--all` call, for example.
  Isolating the hanging test, as proposed, leaves that exposure in place.
  The amended entry names widening the margin as a way to fix the flake, and the diff removes the defect that made the margin 3 s.
