```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test that fails if it breaks.

**Per behaviour, the test that pins it**

1. **The `HangsOnCue` stand-in hangs when `testMapHangingEnv` names a file** (`testmap_integration_test.go:303-309`).
   - `AnAllCallMapsEveryPackageWithATestAndRecordsAFailedBuild` (`:1064-1072`, `:1081`) pins it. If the test did not hang, `TimedOut`, `FailedTests` and the alpha map's `OutcomeTimedOut` entry would mismatch.
   - `ACallEndedDuringABuildOrATestWritesNothing/WhileATestHangs` (`:1429`, `:1442-1445`) also pins it. If the stand-in never wrote the cue file, `pollUntil` would fail after 30 s.
2. **The `HangsOnCue` stand-in passes without the cue**, falling out of the switch into the profile write and `--- PASS` (`:303-309`, `:360-382`).
   - `NamedPackageIsRebuiltWhenFresh` (`:1139`) expects `{Built: [alpha]}` with exit 0, so a hang or a failure would add `TimedOut`, `FailedTests` or `Exit: 1` and the comparison would fail.
   - `AMapRunBuildsNoLym` (`:1336`) pins it the same way.
3. **`testMapSecondUnit` is 500 ms, a 10 s test limit**, in every map call of these tests (`:653`, `:663`, `:818`, `:923`, `:1287`, `:1414`, `:1580`). No test fails if the limit shrinks back to 3 s on a quiet machine. This is the margin the parent's decision "No test seam: the flake is in the test's own time limit" names as untestable, so I raise no finding.
4. **`callCounts` takes an environment parameter** (`:932-935`). Its one caller (`:1064`) passes the cue environment, and the `TimedOut` assertion at `:1069` pins that the parameter reaches the call.
5. **The changed expectations and comments** (`:1139`, `:1231`, `:1336`, `:838`, `:857-858`, `:659-662`) are themselves the assertions above, or comment prose that needs no test.

**Findings:** none.

**Tool calls:** none.
