```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the reedengine seam move, the battenrecipe fold and the audit note.

**Per-function list**

- `New` (`lock.go:76`): sets `executablePath: os.Executable`. A fresh engine resolves the real path, so production behavior is unchanged. Correct.
- `composePaneLaunchLine` (`panebin.go:61`): ran it with a lookup that returns an error. It logs `logger.Warn`, drops the prelude and returns `exe == ""`. The parameter shadows nothing, since the package variable is gone. Correct.
- `composeWindowCommand` (`panebin.go:76`): same unresolvable-path case. It drops the prelude and logs a warning. Correct.
- `composeSwitchCommand` and `composeFollowCommand` (`panebin.go:97,122`): an unresolvable path returns `("", false)` with a warning. `switchBindingCommandsLocked` (`bindings.go:199-203`) turns that into `"", "", ""`, so no binding is pinned. Correct.
- Call sites `spawn.go:118`, `window.go:98` and `bindings.go:199-201`: each passes `e.executablePath`, and `New` always sets it. A nil lookup would panic only for an `Engine` built without `New`, and `New` is the only constructor. Correct.
- `addEnvProbeStrand` (`naming_integration_test.go:22`): it now sets `e.executablePath` on the passed engine. `TestNaming` stays serial, since it uses `logcapture`. Correct.
- `TestComposePaneLaunchLine` (`panebin_test.go`): the local `executablePath` closure is built per subtest and not shared. The tier0 test calls `logcapture.CaptureVerbose` and has no `t.Parallel`. Correct.
- Parallel tier2 tests (`bar`, `bindings`, `contract`, `launchscript`, `ownwindow`, `window`, `windoworder`, `switch`, `tmuxconf`, `revive` and others): none uses `t.Setenv`, `logcapture` or `t.Chdir`. A grep of the `*_integration_test.go` files shows those calls only in `ensuresession`, `serverconfig`, `naming` and `follow`. Each of those four carries a comment naming the state. Correct.
- `TestBoot_ReadsNoOperatorConfig…` (`serverconfig_integration_test.go:76`) uses `t.Setenv("HOME")`, stays serial and has a comment. Correct.
- `TestFollow_RealTmux` (`follow_integration_test.go:20`): serial, comment names the logger. Correct.
- `TestOpenWindow` (`window_test.go`): the parent calls `t.Parallel`, the engine is built per subtest, and the lookup is set per engine. `TestEnsureShellWindow` stays serial because of `logcapture` and sets the lookup per engine. Correct.
- `TestLaunchStrandLocked_WriteFailureSendsTheFullLine` and `TestLaunchStrandLocked_SplitWindowAndPaneTitleCalls` (`spawn_test.go`): both set the lookup on their own engine and stay serial. They carry no comment, but they use no process-global state (see the first finding).
- `TestNameRunShed` fold (`recipe_test.go`): the merged test asserts `NameRunShed == "Run-Shed"` and the row order. The one deleted assertion is kept, and the row loop it duplicated is gone. Correct.
- Audit note (`test-suite-timing.md`): the counts of 14, 3 and 9 match the decision record, which I took as given without re-counting. Correct.

**Findings**

1. LOW, `consistency`: `spawn_test.go:545` (`TestLaunchStrandLocked_WriteFailureSendsTheFullLine`) and `spawn_test.go:~700` (`TestLaunchStrandLocked_SplitWindowAndPaneTitleCalls`) are serial without a comment naming process-global state. After this change they touch none, since the only blocker was the removed package variable. D3 says every such test calls `t.Parallel`, so they should either do so or say why they don't. This is a missed parallelization, not a defect in behavior.

2. LOW, `design`: `TestOpenWindow` is parallel, and `TestEnsureShellWindow` is serial for `logcapture`. This one is not a finding, because `TestEnsureShellWindow` has its comment. I listed it only to show I checked it.

I found no logic or error-path defects in the production code.

**Tool calls**
1. `grep` for `t.Setenv`, `logcapture`, `t.Chdir` and `os.Setenv` in the `*_integration_test.go` files and `enginefixture_helper_test.go`, plus `func Test` and `Parallel` in `follow_integration_test.go`. My context did not show the whole-file contents of these tests, so I confirmed there that the parallel tests touch no process-global state.
