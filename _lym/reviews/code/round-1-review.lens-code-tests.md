```yaml
lens: code-tests
outcome: reported
```

Task: review the diff's tests through the code-tests lens.

**Behaviours and the tests that pin them**

1. Slack state line reads `--dir #{@lym_orch_dir}`: pinned by `TestComposeSwitchCommand`, rows "slack state/posix" and "slack state/pwsh" (`internal/reedengine/panebin_test.go:504-526`).
2. `@lym_orch_dir` is the anchor for the prime and a beam and `''` for a task, set in the same invocation as the other row options: pinned by `TestPinButtonRowLocked` (`internal/reedengine/buttonrow_test.go:264-283`) and by `wantButtonRowArgv` in `TestPinGeometryOptionsLocked` (`internal/reedengine/windowsize_test.go:36-48`).
3. An empty `--dir` (a task's `''`) prints only the away word: pinned by the existing `slackcli` bar test, row `NoDirDrawsOnlyTheAwayWord` (`internal/slackcli/toggle_test.go:~53`).
4. The RC button is drawn on `@strand` together with `@lym_rc_click_command`, lit on `@lym_rc`, and the state job is gone: pinned by `TestStatusFormatActions_TogglesFollowTheFollowButton`, row "RC" (`internal/reedengine/bar_test.go:488-512`).
5. The RC click line is pinned on its own, and no RC state line is pinned or unset: pinned by `TestPinBindingsLocked_PinsTheToggleLinesOnlyWhenEverythingResolves` (`bindings_test.go:284-290`), by `TestPinGeometryOptionsLocked`'s tail and counts (`windowsize_test.go:90,117,275-276,334`), and by `TestAttachArgv_PreflightOnAKnownGoodSession`'s count (`attach_test.go:249`).
6. `clientAgentPaneVia` sends its format and parses the reply into fields, with errors for a tmux failure, short output or an empty pane id: pinned by `TestClientAgentPaneVia` (`internal/reedengine/remotecontrol_test.go:686-733`).
7. `markPaneRemoteControlOnVia` sends its argv, and a tmux error comes back naming the pane: pinned by `TestMarkPaneRemoteControlOnVia` (`remotecontrol_test.go:735-768`).
8. `Engine.MarkRemoteControlOn` marks the strand's own pane and refuses an unknown guid: pinned by `TestMarkRemoteControlOn` (`remotecontrol_test.go:770-809`).
9. `turnOnRemoteControl` handles the orch, a non-orch agent, no agent, an already-marked pane, an unparseable name, a failed inject that marks nothing, and a failed mark that warns: pinned by `TestTurnOnRemoteControl` (`internal/orchcli/remotecontrol_test.go:101-183`).
10. A launch with a name marks the new guid, and every row without a name marks nothing: pinned by `TestStart_LaunchPassesAndMarksRemoteControl` (`remotecontrol_test.go:46-97`).
11. Adopting a live strand marks nothing: pinned by `TestStart_LiveStrand` (`start_test.go:758-760`).
12. In the `rc-on` RunE, a `ClientAgentPane` failure is reported through `ShowToggleOutcome` and `output.Err`, and the mark closure targets `pane.PaneID`: **no test**.
13. A failed launch mark is logged at Warn and never fails the launch (`markLaunchedRemoteControl`'s error branch): **no test**. No row of `TestStart_LaunchPassesAndMarksRemoteControl` makes `markRemoteControl` return an error.

**Findings**

- **MEDIUM**, design. `internal/orchcli/remotecontrol.go:159-181` (rc-on RunE): no test covers the click verb's new wiring.
  - A `ClientAgentPane` error has to reach `ShowToggleOutcome` and exit with `output.Err`.
  - The mark has to land on the clicked pane's id.
  - The grep over `*_test.go` finds `rc-on` only in `panebin_test.go:521,529`, the composed line, and finds no test that calls `ClientAgentPane` or runs the verb.
- **LOW**, design. `internal/orchcli/remotecontrol.go:94-96` and `start.go:679-681`: no test covers the "failed mark never fails the launch" branch.
  - Every `markRemoteControl` fake in `remotecontrol_test.go:77-80` returns nil.
  - Adding a row whose mark fails and asserting that the launch still succeeds would pin it.
- **LOW**, design (test economy). `TestMarkPaneRemoteControlOnVia/SetsThePaneOption` (`internal/reedengine/remotecontrol_test.go:742-754`) duplicates `TestMarkRemoteControlOn/MarksTheStrandsPane` (`remotecontrol_test.go:781`).
  - Both assert the identical argv `set-option -p -t %7 @lym_rc 1`.
  - Folding it into `TestMarkRemoteControlOn` loses nothing.
  - The `ATmuxFailureIsReturned` subtest pins the error message, so keep it.

Tool calls:
1. Grepped `*_test.go` for `rc-on`, `remoteControlOnVerb` and `ClientAgentPane`, and read `slackcli/toggle_test.go:60-100`: the context did not show whether any test drives the rc-on verb or how the slack bar handles an empty dir.
2. Read the row names at `slackcli/toggle_test.go:30-60`: to confirm a no-dir row exists for the task session's `''` word.
