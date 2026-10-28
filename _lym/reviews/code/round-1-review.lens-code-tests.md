```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, name the test that pins it, or none.

**Per behaviour: pinning test**

| Behaviour | Pinned by |
|---|---|
| Pane launch line exports both build-server variables, on every dialect | `TestComposePaneLaunchLine` (`panebin_test.go`) pins order and `Contains` per dialect. `TestLaunchStrandLocked_LaunchScript` (`spawn_test.go`) pins the script content. |
| Exports survive an executable-lookup failure | `TestComposePaneLaunchLine` rows `ExecutableErrorPassesTheCommandThrough`, `ExecutableErrorWithEmptyCommandIsTheBuildServerExportsAlone` and `ExportsSurviveAnExecutableError`. |
| A tier's command sees both variables, overriding inherited values | The `TestGateTest_Scenario` "a free slot runs the tier..." row sets the inherited values `0` and `1` and asserts `1` and `0`. |
| Build, vet and guard runs get both variables | None. Only the tier stand-in's record is asserted. |
| A tier's or build's own `env` can override the variables (auto-mode assumption, stated in `docs/dotnet-hub.md`) | None. |
| An unset tier or build `dir` follows the anchor; `dir: .` stays at the worktree root | `TestGateTest_UnsetDirFollowsTheAnchor` runs the build and tier0 in the anchor and tier6 (`dir: "."`) at the root. |
| An unset `dir` stays empty after config load | The `config_test.go` expectations changed from `Dir: "."` to empty. |
| A set `dir` such as `sub` stays root-relative under a subdirectory anchor | None. `.` is the only set value in the new test, and `sub` appears only in a `config_test.go` load row. |
| A root anchor yields `.`, so the green-record module key and tier fingerprints are unchanged | Only the existing integration scenarios, which run with a root anchor. No assertion names it. |
| `WithDefaultDir` leaves a declared-empty build (no command) alone | None, and no direct unit test of the method. The integration test covers only the fill path. |
| Retry fires only on a failed, untimed-out, uncancelled run with a marker, once, and the retry's result is the tier's | `TestRetryOnShortage` rows `PassIsNotRerun`, `FailureWithoutMarkerIsNotRerun`, `TimeoutIsNotRerun`, `RefusalIsNotRerun`, `CancelledCallIsNotRerun`, `ShortageThenPassEndsPassed`, `ShortageThenOrdinaryFailureCarriesNoShortage` and `ShortageTwiceCarriesTheMarker`. |
| Markers match case-insensitively; all three markers are recognised | `TestRetryOnShortage`: `Resource temporarily unavailable`, `failed to create new OS thread` and `Cannot fork` are all capitalised in the logs. |
| The stderr announcement, and the retried log opening with it | `TestRetryOnShortage`, the `wantRetry` assertions. |
| The retry delay is injected | `TestRetryOnShortage` passes delay 0. |
| The retry is logged through `logger.Warn` | None. |
| A ctx cancelled during the delay returns the first result | None. Only the cancelled-before-start row exists. |
| The closing line labels a twice-failed marker run with the way forward | `TestFailedRunLabel`. |
| `runGreenAware` actually wraps `runTier` with `retryOnShortage`; `shortage` flows `report` → `runEnd` → `failedRunLabel`; the real `shortageRetryDelay` is set | None. No integration test makes a stand-in emit a marker, so removing the wrapper or the `shortage:` propagation fails nothing. |
| `FailedRunsPrefix` is still the text `printFailedRuns` prints | No test I can confirm. `TestVerifyFailureFindings` uses the constant itself, so the verify-findings test would not catch a changed prefix. |
| Verify findings list failures, with output nested under the header or above it, plus the closing line, and no log tail | The new `TestVerifyFailureFindings` row covers `TestA` (output above the header) and `TestC` (nested output), the closing line, and `wantNot` for the tail. |
| Verify findings keep the log tail when the parser names no failure | The existing rows, now asserting `Log tail:` and `last line`. |
| Verify findings keep the log tail when the parser returns only an opaque failure | None. Those rows probably produce no failure at all. |
| `verifyFailuresBytes` bound: "More failures are in the full log." and the first failure kept whole | None. |
| `verifyOutputAboveLines` cap of 40 lines, and stopping at the nearest `=== ` or `--- ` marker | The stop at `--- ` is exercised by the `TestA` case only. The cap is not tested. |

**Findings**

1. **MEDIUM, design.** `internal/gatecli/run.go:426` (`runGreenAware`) and `run.go:482` (`report`). The retry wiring has no covering test, so the acceptance criteria "retry passes → tier ends passed with the retry line on stderr" and "marker twice → closing line names a resource shortage" are pinned only at unit level. `retryOnShortage` is tested over fake attempts and `failedRunLabel` over a hand-built `runEnd`. Nothing drives `runTest` end to end. Dropping the `retryOnShortage` call, the `shortage: result.shortage` propagation or the `failedRunLabel` call at `run.go:159` would fail no test. Fix: add one `TestGateTest_Scenario` row using the stand-in with a marker in its output, covering the retry-passes and fails-twice cases, with `shortageRetryDelay` set to 0 in the test request.

2. **LOW, design.** `internal/gatecli/run.go:127`, `withBuildServerOff`. Only the tier run's environment is asserted. The build step, vet and guard runs share the same `env`, but the decision record requires "every tier, build, vet and guard run", and the build run's variables are not asserted. Fix: have the stand-in's build mode record the variables too and assert them in the existing build scenario.

3. **LOW, design.** `internal/loomshed/verifygate.go:439`. The byte bound has no row: no log with failures past `verifyFailuresBytes`, so the "More failures..." break and the keep-the-first-failure-whole rule are unpinned. Fix: add a `TestVerifyFailureFindings` row with enough failing tests to exceed 8192 bytes. Assert the first is present, the later ones are cut, and the "More failures" line appears.

4. **LOW, design.** `internal/gatecli/tierrun.go:226`. The cancel-during-delay branch (`ctx.Done()` while the timer waits) is untested. Only the already-cancelled case is covered, and that one returns before the timer. Fix: add a `TestRetryOnShortage` case with a positive delay and a ctx cancelled while it waits, asserting that one attempt ran and the first result came back.

5. **LOW, design.** Documented but unpinned. A tier's own `env` overriding the build-server variables is in `docs/dotnet-hub.md`, and a set `dir` under a subdirectory anchor (such as `sub`) stays root-relative. Fix: extend `TestGateTest_UnsetDirFollowsTheAnchor` with a tier whose `dir` is a subdirectory, and the env scenario with a tier `env` that sets `MSBUILDDISABLENODEREUSE`.

No test redundancy found. The new tests cover distinct behaviours, and the `config_test.go` and `panebin_test.go` edits are expectation updates, not new tests.

**Tool calls:** none. My context already held every diff and test I needed.
