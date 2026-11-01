```yaml
lens: row-validity
outcome: reported
```

Row-validity lens review of the nine plan cards.

1. **BLOCKING — Card 2, desktop `TestLoadConfig`, mapping-group row.** The row expecting a machine override `buttons: {task: {label: x}}` to fail naming `buttons.task` and "different shape" would fail as written.
   - `yamlengine.LeafPaths` steps into a non-empty mapping, so the unit is `buttons.task.label`.
   - `nodeAt` returns nil once it meets the template's sequence (`internal/configengine/layers.go:392-415`).
   - `ValidateMachine` therefore returns `key "buttons.task.label" is not in the desktop template` (`layers.go:341-343`), not the shape error.
   - Only the empty-mapping row `buttons: {task: {}}` gets "different shape".
   - Fix: expect "not in the desktop template" for the non-empty mapping row.

2. **MEDIUM — Card 2, `TestLoadConfig_TemplateDefaultsResolve`, local-override row.** The row copies `LocalExecutableKeyIsRefused`, which asserts the quoted key `"shell"` (`internal/reedengine/config_test.go`). The planned row expects `"buttons"`.
   - The local file `buttons: {task: []}` has the unit path `buttons.task`.
   - `ValidateLocal`'s message is `key "buttons.task" is executable…` (`layers.go:298-300`), which never contains `"buttons"` with its closing quote.
   - Fix: assert `"buttons.task"`.

3. **MEDIUM — Cards 3 and 4, Warn assertions in parallel tests.** These rows read the log through `logcapture` inside `t.Parallel` tests.
   - Card 3's `TestPlaceButtons` and card 4's new `TestReedGeometry` rows (a `{` record, and a machine `buttons: {task: {}}` with "the Warn logged") both do this; `TestReedGeometry` calls `t.Parallel` today.
   - `logcapture.Capture` repoints the global logger output and restores it in Cleanup (`internal/testkit/logcapture/logcapture.go:42-55`). Parallel siblings stomp on each other's capture.
   - `PATTERN-test-speed` "Tests run in parallel" says a test touching process-global state drops `t.Parallel` and says so.
   - Fix: put the Warn-asserting rows in a non-parallel test or subtest, the way `TestPinGeometryOptionsLocked` does, or drop the Warn assertions.

4. **MEDIUM — Card 9, `TestPinGeometryOptionsLocked`, `ShellPinsFollowTheOptionPins`.** Card 9 adds the `set-option -gu @lym_button_command` unset after the follow-command unset. The subtest's `wantLast`, which is the follow unset then the build-stamp unset, then no longer matches the last calls.
   - The card names only `wantOptions`, the pin counts and `BindingsAreIssuedAfterTheOptionPins`; the `wantLast` and `wantTail` slicing is unlisted (`internal/reedengine/windowsize_test.go` `ShellPinsFollowTheOptionPins`).
   - Fix: state that `wantLast` gains the button-command unset between the two.

5. **MEDIUM — Card 5, `TestBar_LineTwoListsSessionsPrimeFirst`.** Once `follow` leaves line 2, the helper `assertSessionButtonsOnce` slices `line[:firstFollow]` with `firstFollow == -1` and panics (`internal/reedengine/bar_integration_test.go:229-232`).
   - The card says the follow rows move to line 3 but never names this helper's slice as needing a rewrite.
   - Fix: name the helper's no-comma check and give it its new anchor, `nolist align=right`.

6. **MEDIUM — Card 8, `TestRunButtonVia`, two unreached branches.**
   - The prescribed "unresolvable executable on a confirm entry: Warn, runs nothing" branch has no row.
   - Step 7's spawn-failure message (`deps.run` returning an error) has no row; only the non-zero exit is covered.
   - Fix: add both rows; the stub `executablePath` and stub `run` can already produce both states.

7. **LOW — Card 8, `TestRunButtonVia`, helper not listed.** The test drives a `TmuxCmd` built by `newTmuxCmdForSocketPath`, which needs `installFakeTmuxOn`, as `internal/reedengine/follow_test.go:117-118` does. The card's `Uses` lists only `installFakeTmux`.
   - Fix: add `internal/reedengine#installFakeTmuxOn` to `Uses`.

8. **LOW — Card 7, empty strand name.** The prescribed "an empty name unsets `@lym_strand_name`" branch has no row.
   - Every `TestLaunchStrandLocked_SplitWindowAndPaneTitleCalls` row only gains the set call.
   - Fix: add a row with an unnamed strand that expects `set-option -p -u -t %new @lym_strand_name`.

9. **LOW — Card 9, `TestBindings_ButtonClickRunsTheVerbInTheRunDirectory`, stub place unspecified.** The row says the stub gives "a temp run directory", but card 6 makes the run directory `geom.AnchorPath` for prime and beam. Only a task place with a stub `Beam` returning that temp `Anchor` yields it.
   - Fix: state the stub's `Place` is task and its `Beam` returns the temp directory, so the recorded `pwd` assertion pins the run-directory rule.

10. **LOW — Card 8, `TestRunButtonVia`, vague row.** The row "a `#` in a label-free message text is doubled" names no input that yields such a text; every message the card defines carries a label or fixed text.
    - Fix: name the input, for example a label `a#b` in the changed-buttons message, or drop the row.

Tool calls:
- Read `internal/shell/shell_test.go`'s test list and the re-pin row of `TestBindingsBuildStamp`, to confirm `TestShell_Exit` exists and how the re-pin row counts calls.
- Read `TestAttachArgv_PreflightOnAKnownGoodSession`'s count, `TestComposeSwitchCommand` and `fabricengine.ConfigDirs`, to check card 5–9's counts, card 9's subtest shape and card 4's machine-layer dirs.
- Read the `logcapture` capture functions, `configengine` `nodeAt` and follow_test's fake install, for findings 1, 3 and 7.
- Read `logcapture`'s function list and the `FillMissing` error wrap, to confirm the fill error is `ErrInvalid` and that `Capture` swaps the global output.
- Read the `LoadOrTemplate` fill site in `internal/configengine/config.go:230-240`, to confirm card 2's `buttons: null` row reaches `ErrInvalid`.
