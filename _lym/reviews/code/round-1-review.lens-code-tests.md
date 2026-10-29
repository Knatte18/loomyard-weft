```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff changes, the test that pins it, or none.

**Per behaviour**
- **`New` sets `executablePath` to `os.Executable`:** no direct assertion. It is pinned only indirectly, because a nil field would panic in the integration engines' `Up()` and in `launchStrandLocked`. Tests always overwrite the field after `New`. Not raised as a finding.
- **`composePaneLaunchLine` with an injected lookup, resolvable and unresolvable:** `TestComposePaneLaunchLine` (`panebin_test.go:157`).
- **`composeSwitchCommand` and `composeFollowCommand`, resolvable and unresolvable with the `logger.Warn`:** `TestComposeSwitchCommand` and its subtests (`panebin_test.go:294-339`).
- **`composeWindowCommand` carries the path:** `TestOpenWindow` asserts `wantCommand` with the stub path (`window_test.go:139`).
- **`composeWindowCommand` with an unresolvable path (drops the prelude and warns):** none. A grep of `window_test.go` and `panebin_test.go` finds no unresolvable-lookup case for it.
- **`e.executablePath` wired into `launchStrandLocked`:** `TestLaunchStrandLocked_LaunchScript` (`spawn_test.go:451-483`) and the integration test `TestLaunchScript_SourcedScriptRunsInThePaneShellScope`.
- **`e.executablePath` wired into the key bindings:** `bindings_integration_test.go:56`, which injects the recorder script and asserts the Alt-key and follow commands run it.
- **`t.Parallel` on the tests that touch no process-global state:** by construction, with no assertion row. The serial tests `TestEnsureSession_BootedTrueOnColdSessionFalseOnWarm`, `TestAddStrand_LogsAttributionOnlyOnColdBoot`, `TestBoot_ReadsNoOperatorConfigAndStartsNonLoginPanes`, `TestNaming`, `TestFollow_RealTmux` and `TestEnsureShellWindow` each carry a comment naming the state.
- **battenrecipe merge:** `TestNew_RowNamesMatchTheDurableIdentityConstants` now asserts both `NameRunShed` and the row order.

**Findings**

1. **MEDIUM, consistency.** `internal/reedengine/spawn_test.go:545` (`TestLaunchStrandLocked_WriteFailureSendsTheFullLine`) is serial and calls `logcapture.CaptureVerbose`, which is process-global state. It carries no comment saying so. Decision-record D3 and the acceptance criterion require every test that does not call `t.Parallel` to name that state. Evidence: `spawn_test.go:541-545`, plus `logcapture` at its line 17 of the sampled range, about `spawn_test.go:556`.
2. **MEDIUM, consistency.** `internal/reedengine/spawn_test.go:645` (`TestLaunchStrandLocked_SplitWindowAndPaneTitleCalls`) is serial with no comment naming the state, only a `//testtiming:keep` line. I did not confirm whether the whole test uses `logcapture`. The subtest range around `spawn_test.go:757` calls `logcapture.Capture`. If it does, the same D3 gap applies. If it does not, it should call `t.Parallel`, since its serial status came only from the removed seam. Evidence: `spawn_test.go:639-645`.
3. **LOW, scope.** Decision-record D2's acceptance criterion says an unresolvable path still drops the prelude or the binding, "each asserted by the existing tests". The window command's unresolvable-lookup branch (`composeWindowCommand`) has no such test. The diff only changed its signature, so the gap may predate it, but the criterion is not met. Evidence: no match for an unresolvable case in `window_test.go` or `panebin_test.go` (grep).

**Tool calls**
1. Read `follow_integration_test.go` for its parallel and comment markers, plus a grep of the `spawn_test.go` ranges, because the diff showed no context for them.
2. Read the comments above the two `spawn_test.go` serial tests and grepped for unresolvable-lookup cases in the window tests, because the diff context did not show them.
