```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I found the test row that would fail if it broke.

**Behaviour → test that pins it**
1. The claude launch line carries `--remote-control '<name>'`, quoted in posix: `TestBuildLaunchCmd` row `remote_control_posix`.
2. The claude resume line carries the flag, in pwsh: `TestBuildResumeCmd` row `remote_control_on_resume`.
3. An empty name adds no flag: the existing rows of `TestBuildLaunchCmd` and `TestBuildResumeCmd`, now called with `""`.
4. `Claude.RemoteControlSequence` returns the text and an Enter: `TestRemoteControlSequence`.
5. `Runner.TurnOnRemoteControl` pastes the sequence through the guarded path, refuses `\r\n`, types nothing on an engine without the capability, and returns `toldErr`: **none**.
6. Creel validates `remote-control`, refusing an empty argument and a control character: `TestStore_InjectAndHold`, two new rows.
7. Creel's unknown-command message lists `remote-control`: none. This is error text with no behaviour behind it; not a gap.
8. Delivery maps `remote-control` to `TurnOnRemoteControl(guid, name)`: `TestHubDelivery`, new row.
9. orch.yaml's template sets the switch on with `{machine}::{name}`: `TestLoadConfig_RemoteControl` row "the template is on…".
10. `LoadConfig` refuses an empty name while the switch is on, refuses a control character, and loads switch-off with an empty name: `TestLoadConfig_RemoteControl`, three rows.
11. `RemoteControlSessionName` fills the placeholders in one pass, leaves other brace text literal, and its default equals `FormatMachineAddress`: `TestConfig_RemoteControlSessionName`.
12. `launchRemoteControlName`:
    - For a listed strand with the switch on, off, or on an engine without the capability: `TestStart_LaunchPassesAndRecordsRemoteControl`.
    - For an unlisted strand, through `FixedStrandName`: **none**.
    - When the machine name fails, it should log and launch without the flag: **none**.
13. `launchOrch` passes the name on the resume branch (`st.SessionID != ""`): **none**. The start harness only exercises the fresh launch (`internal/orchcli/start_test.go:179`, no SessionID seeded).
14. The relaunch records on or off for the new guid: `TestStart_LaunchPassesAndRecordsRemoteControl`, which asserts `RemoteControlOn(new-guid)`.
15. `recordRemoteControl` logs a write failure and does not fail the launch: none. This is a log-only path; not a gap.
16. `turnOnRemoteControl` queues and records, and refuses with no record for: no live orch, no orch, an engine without remote control, and a failed inject: `TestTurnOnRemoteControl`. Its machine-name error path has no row, but `remoteControlSessionName` returns that error unchanged.
17. `injectRemoteControlCommand` signs as program `REMOTECONTROL`: none. It is wiring to a real creel client and is covered by the program-name grammar elsewhere.
18. `rc-bar` skips the orch wiring in `PersistentPreRunE`, and `remoteControlBarWord(dir)` gives an empty word for an empty or unresolvable dir: **none**.
19. `remoteControlBarWordAt`:
    - No state, no strand, no record, on, off, or on for an old strand: `TestRemoteControlBarWordAt`.
    - An unreadable record should print as rc-off: no row.
20. `RemoteControlOn` and `WriteRemoteControl`:
    - Absent, on for its strand, on for another strand, off: `TestRemoteControlOn`.
    - Undecodable JSON returns an error: `TestRemoteControlOn`.
21. The `rc-on` verb shows the outcome through `ShowToggleOutcome` and sets the exit envelope: none. This is the same shape as the slack toggle verb; not a gap.
22. Status bar: the RC button's position, its shown condition and lit on `rc-on`: `TestStatusFormatActions_TogglesFollowTheFollowButton`, RC row.
23. `bindingArgvs` RC branch: bound only when pinned, and dispatching on range `rc`: the `TestBindingArgvs_…` rows "RCOnly", "All" and "None", plus the separator count.
24. `pinBindingsLocked` pins or unsets both RC options, and the click holds the RC branch only when everything resolves: `TestPinBindingsLocked_PinsTheToggleLinesOnlyWhenEverythingResolves`.
25. `composeRemoteControlClickCommand` and `composeRemoteControlStateCommand`: exact lines in posix and pwsh, and an unresolvable executable warns and pins nothing: `TestComposeSwitchCommand`, the rc rows.
26. The geometry pins unset the RC options, and the set-option counts: `TestPinGeometryOptionsLocked`, `TestAttachArgv_PreflightOnAKnownGoodSession`.
27. The `bindingLinesLocked` refactor keeps the switch, follow and button lines when the Slack or RC lines fail: `TestPinBindingsLocked_…`, test-binary row ("pins no button line").
28. The `FixedStrandName` extraction from `StrandOf` changes no behaviour: the existing callers' tests. No orchroute test file names it, and the refactor preserves behaviour.

**Findings**

- **MEDIUM | design | `internal/shuttleengine/session.go:1211-1231` (`Runner.TurnOnRemoteControl`)**
  This new runner method has no test, unlike its peers.
  - Peers are pinned by `checkPastedSessionCommand` (`internal/shuttleengine/session_test.go:886`) and the dispatch table (`internal/shuttleengine/enginedispatch_test.go:262`).
  - Uncovered behaviour: the pasted `/remote-control <name>` through the guarded path, the refusal of a name containing `\r` or `\n`, an engine without `RemoteController` typing nothing, and the `toldErr` short-circuit.
  - The only test touching it is `TestHubDelivery`, which reaches a recording fake (`internal/creelcli/delivery_test.go:104`).
  - Fix: add rows to `checkPastedSessionCommand` and the dispatch table, plus a refusal row for a multi-line name.

- **LOW | design | `internal/orchcli/remotecontrol.go:255-274` (`launchRemoteControlName`)**
  The unlisted-strand branch and the name-formation failure branch have no row.
  - Unlisted strand: `FixedStrandName` should form `<sn>:orch`; on failure the session should launch without the flag.
  - Name formation failure (machine name error, unparsable name): it should log at Warn and return `""`, so the launch still succeeds without the flag.
  - `TestStart_LaunchPassesAndRecordsRemoteControl` only seeds a listed strand (`internal/orchcli/remotecontrol_test.go:96`) and a machine name that succeeds (line 98).
  - Fix: add rows `NoListedStrand` and `MachineNameUnreadable` to that test.

- **LOW | design | `internal/orchcli/start.go:717` (resume branch of `launchOrch`)**
  No row asserts that a resumed launch carries `RemoteControlName`. The harness seeds no `SessionID` (`internal/orchcli/start_test.go:179`), so only the fresh launch at line 734 is exercised.
  - Fix: add a row to `TestStart_LaunchPassesAndRecordsRemoteControl` that seeds a state with a SessionID.

- **LOW | design | `internal/orchcli/remotecontrol.go:325-361` (`rc-bar` verb and `remoteControlBarWord`)**
  Nothing pins three behaviours:
  - the verb skips the orch wiring (`internal/orchcli/cli.go:122`), so it prints rather than refuses outside a prime or beam;
  - it prints an empty line for an empty or unresolvable `--dir`;
  - an unreadable record reads as rc-off (`RemoteControlOn` error leads to Unlit, line 371).

  `TestRemoteControlBarWordAt` covers only `remoteControlBarWordAt` with readable records.
  - Fix: add an `UnreadableRecord` row to `TestRemoteControlBarWordAt`, and one command-level row running `orch rc-bar --dir <non-worktree>` through the cobra tree, expecting empty output and exit 0.

Tool calls:
1. Grepped the test files for `TurnOnRemoteControl`, `FixedStrandName`, `rc-bar`, `remoteControlBarWord` and `machineName`, plus the start harness's state seeding: the manifest held only the diff's tests, not the existing test suite.
2. Grepped for the peer runner command tests (`ReloadPlugins`, `TypeColor`), the start harness's starter seed and orchroute's `StrandOf` tests: I needed the existing test patterns, which the manifest lacked.
