```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I ran each branch and function the diff adds or changes, step by step.

**Per branch: the case I ran, and whether it is correct**

- **`runTestMapStandIn`, `HangsOnCue` case, cue unset** (`testmap_integration_test.go:303-309`): `TestAlphaHangsOnCue` with an empty `testMapHangingEnv`. The `if` is skipped and the switch exits. The profile marks block 0 covered and block 1 not, then prints `--- PASS` and exits 0. Correct.
- **Same case, cue set and the append fails**: it returns 1 and prints no FAIL line, so the run is recorded as failed. The `Hangs` case behaves the same at lines 311-313. Correct.
- **Same case, cue set and the append succeeds**: it sleeps an hour, so the run ends on the 10 s limit as timed out. Correct.
- **Suffix order**: a name ending in `HangsOnCue` does not end in `Hangs`, so the order of the two cases cannot change which one is taken. The cases before it (tier env missing, `Fails`, `Skips`) do not match this name. Correct.
- **`callCounts` with the new `env` parameter** (line 932): its only caller (line 1064) passes `withEnv(testMapHangingEnv, …)`. `withEnv` clones `environ`, so the shared environment is not changed for later subtests. Correct.
- **`AnAllCall…` with the cue set** (lines 1062-1133): tier1 runs `TestAlphaTwo` and `TestAlphaHangsOnCue` side by side with parallel 2.
  - The hanging test times out at 10 s and `TestAlphaTwo` passes.
  - `runBatch` (`testmappackage.go:296-309`) reruns only the timed-out test alone. The cue is still set, so it hangs again and times out again.
  - It lands in both `timedOut` and the failed tests, and there is no `passed_alone` entry.
  - The map entry is `OutcomeTimedOut` at tier 1, and `Tests: 3` for alpha still holds.
  - Correct.
- **`NamedPackageIsRebuiltWhenFresh` with no cue** (lines 1136-1147): every alpha test passes, so there is no rerun and nothing timed out. Exit 0 with `Built: [alpha]` matches. Writing the index does not depend on the exit code. Correct.
- **`AnEllipsisTargetMapsEveryPackageBelowIt`** (lines 1225-1235): exit 1 still holds through `outcomes.TestOutcomesFails`, broken's failed build, and zmoved moving HEAD again. The new comment leaves zmoved out, as the old one did. The assertion is correct.
- **`AMapRunBuildsNoLym` with no cue** (lines 1319-1352): exit 0 and `Built: [alpha steady]`. The seen file gets one empty line per run, four runs, so `len(paths) &gt;= 2` and every path is empty. Correct.
- **`ACallEndedDuring…/WhileATestHangs`** (lines 1429-1458): the cue is set, so `HangsOnCue` appends to the started file and then hangs. The poll sees the file and the cancel fires before the 20 s limit, which is 20 × `time.Second`. Exit 1 says the call was cancelled, and `BuiltAt` is unchanged. Correct.
- **The other two rows, `WhileTheGoListHangs` and `WhileTheCoverageBuildHangs`**: they hang in `go list` or in the build, before any test runs, so the missing cue does not matter. Correct.
- **`testMapSecondUnit`, 500 ms** (line 663): it reaches only `testTimeout` (`testmap.go:338`), giving 20 × 500 ms = 10 s.
  - `ACallInsideAnEnclosingGateRunsSlotTakesNoSlot` (1 min context, steady) has no hang. Correct.
  - `AMapRunHoldsTheLargerSlotAskOfItsTiers` (2 s context) is cancelled while it waits for a slot, so the test limit never applies. Correct.
  - The `--card` call in `TestGateTestMapInATaskWorktreeKeepsItsOwnMaps` maps solo, which never hangs. Correct.
- **`AComputedScopeTakesNoBaseFromTheGreenRecord`**: only the built sets are compared, and both calls run without the cue. Correct.

**Findings**

None.

Outside my lens: in the `--all` call, `TestAlphaTwo` still runs beside a hanging test, now under a 10 s limit. If its start runs past 10 s, a `passed_alone` entry can still appear there. That is a wider margin, not a deterministic fix, so it is for the handler or the scope lens to weigh against the parent's "deterministic" decision.

Tool calls: none.
