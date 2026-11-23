```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I looked for the test row that fails when that behaviour breaks.

**Per behaviour, the test that pins it**

- `claudeengine.bridgeSessionRecordedIn`:
  - a record past the checkpoint, before it, or exactly at it; a shrunk transcript; a half-written line; a missing file: `TestBridgeSessionRecordedIn` rows `BridgeRecordConnects`, `BridgeRecordBeforeTheCheckpointIsOld`, `BridgeRecordAtTheCheckpointConnects`, `ShrunkTranscriptIsReadWhole`, `BridgeRecordStillBeingWrittenIsNotRead`, `MissingTranscriptRecordsNoConnection`.
  - a policy refusal reads as no connection: row `PolicyRefusalRecordsNoConnection`.
- `claudeengine.isBridgeSessionRecord`:
  - the type match against the record type quoted in a message: row `RecordTypeQuotedInAMessageIsNoConnection`.
  - the `bridgeSessionId != ""` requirement: none.
- `Claude.RemoteControlCheckpoint` (size, missing transcript → 0, stat error) and `Claude.RemoteControlConnected` (path from `sessionProjectDir`): none.
- `Runner.awaitRemoteControl`:
  - connect at once, after ticks, at the deadline (41 reads, 20 s), read error, engine without a reader: `TestRunner_AwaitRemoteControl` rows.
  - empty `sessionID` → false and no read: none.
- `Runner.remoteControlCheckpoint`:
  - the engine's answer, and zero for an engine without a reader: `TestRunner_AwaitRemoteControl`.
  - empty `sessionID` → 0: none.
  - the error wrap on a failed read: none.
- `Runner.RemoteControlCheckpoint` / `AwaitRemoteControl`, dispatch by engine name: `TestRunner_AwaitRemoteControl`.
- `Runner.TurnOnRemoteControl`:
  - reports connected or not: `TestRunner_TurnOnRemoteControl_ReportsTheRecordedConnection`.
  - an engine without `RemoteController` → false: the capability subtest of `TestRunner_TurnOnRemoteControl_PastesTheNamedCommandThroughTheGuardedSend`.
  - the checkpoint is taken before the paste: none in effect (see finding 4).
  - a failed checkpoint read types nothing: none.
  - a failed await after the command was typed returns an error: none.
- creelcli `turnOnRemoteControl`:
  - connected → mark: `TestHubDelivery`, the new `connected` assertion.
  - not connected → no mark: `TestHubDelivery`'s existing remote-control row (recorder `connected` false).
  - `TurnOnRemoteControl` error → returned: none.
  - a failed mark → logged, nil returned: none.
- `sessionCommandOps.MarkRemoteControlOn`, delegation to reed: none (thin delegation).
- orchcli `launchRemoteControlCheckpoint`:
  - fresh launch → 0: `TestStart_LaunchPassesAndMarksRemoteControl` rows `SwitchOn` (`minted-id@0`).
  - resumed session → the checkpoint: `ResumedSessionCountsOnlyPastTheCheckpoint` (`sess-1@42`).
  - unreadable checkpoint → no wait and no mark: `UnreadableCheckpointLeavesThePaneUnmarked`.
- orchcli `markLaunchedRemoteControl`:
  - connected → mark: `SwitchOn`.
  - refused → no mark: `RefusedConnectionLeavesThePaneUnmarked`.
  - a failed mark still launches: `FailedMarkStillLaunches`.
  - checkpoint reset to 0 when the resume fell back to a fresh session (`started.SessionID != resumed`): none.
  - await error → no mark: none.
- orchcli `turnOnRemoteControl` and the `rc-on` verb mark nothing: `TestRemoteControlOnVerb_QueuesForTheClientsPaneWithoutMarkingIt` (no `set-option`) and `TestTurnOnRemoteControl`.
- Removal of reedengine `MarkPaneRemoteControlOn`: nothing to pin; `TestMarkRemoteControlOn` still covers the engine op.

**Findings**

1. LOW, design.
   - Location: `internal/orchcli/remotecontrol.go`, `markLaunchedRemoteControl`, the `if started.SessionID != resumed { checkpoint = 0 }` branch.
   - Summary: no row covers a resume that falls back to a fresh launch with remote control on. Deleting the reset would pass every test, so the fresh session would be read from the old session's offset.
   - Evidence: `internal/orchcli/remotecontrol_test.go` rows (diff 002, lines 311–322) carry no `resumeErr`. `start_test.go:1553-1557` exercises the no-transcript fallback only without remote control.
   - Fix: add a row to `TestStart_LaunchPassesAndMarksRemoteControl` with `recordedSession: "sess-1"`, the harness starter's `resumeErr: noTranscriptErr` and `wantAwait: "minted-id@0"`.
2. LOW, design.
   - Location: `internal/shuttleengine/claudeengine/remotecontrol.go`, `RemoteControlCheckpoint` and `RemoteControlConnected`.
   - Summary: no test covers the checkpoint read. It should return the transcript's size, return 0 for a missing transcript, and surface a stat error. Nothing pins the `&lt;projectDir&gt;/&lt;sessionID&gt;.jsonl` path either method reads.
   - Evidence: a grep of `claudeengine/*_test.go` for `RemoteControlCheckpoint|RemoteControlConnected` finds nothing. The reach report says no mapped test reaches either.
   - Fix: table-test both methods over a fake config dir with a written transcript: the size, a missing file → 0, and a connection found through the same resolved path.
3. LOW, design.
   - Location: `internal/shuttleengine/session.go` `TurnOnRemoteControl`, and `internal/creelcli/delivery.go` `turnOnRemoteControl`.
   - Summary: error paths have no rows:
     - a checkpoint read that fails before the paste: nothing should be typed;
     - an await read that fails after the paste;
     - creel's handling of a `TurnOnRemoteControl` error, and of a failed mark, which should still return nil.
   - Evidence: `remoteControlRecord.RemoteControlCheckpoint` never fails (`remotecontrol_test.go:26-30`). `recordingCommands` returns only `r.connected, nil` and a nil mark error (`delivery_test.go:107-114`).
   - Fix: add a checkpoint-error field to `remoteControlRecord`, with rows asserting no paste and the error. Add error and mark-error fields to `recordingCommands`, with rows on the `TestHubDelivery` command table.
4. LOW, design.
   - Location: `internal/shuttleengine/remotecontrol_test.go:117-157`, `TestRunner_TurnOnRemoteControl_ReportsTheRecordedConnection`.
   - Summary: the doc comment says the test pins the checkpoint the session reached "before the paste". The fake's `checkpoint` is a constant 42, so taking the checkpoint after the paste would pass as well; the ordering is unpinned.
   - Evidence: lines 137 and 150–152 set `record.checkpoint = 42` once and assert `readFrom[0] == 42`.
   - Fix: make the fake's checkpoint grow on paste, for example from a `PasteSubmitCalls`-driven counter, and assert the pre-paste value.
5. NIT, consistency.
   - Location: `internal/shuttleengine/remotecontrol_test.go:117`, `TestRunner_TurnOnRemoteControl_ReportsTheRecordedConnection`.
   - Summary: the redundancy report lists it as contained by `TestRunner_TurnOnRemoteControl_PastesTheNamedCommandThroughTheGuardedSend` and `TestRunner_AwaitRemoteControl`. Neither covering test asserts the `connected` value `TurnOnRemoteControl` returns, or that the read takes the checkpoint TurnOnRemoteControl took, so this test pins what coverage cannot see.
   - Evidence: the PastesTheNamed wrapper discards the bool (`session_test.go:954`); the await test calls `AwaitRemoteControl` directly (`remotecontrol_test.go:96`).
   - Fix: mark it `//testtiming:keep pins the connected result TurnOnRemoteControl returns and the checkpoint its read starts from` directly above `func Test…`.
6. NIT, design.
   - Location: `internal/shuttleengine/claudeengine/remotecontrol.go:99-101`, `isBridgeSessionRecord`.
   - Summary: no row has a `bridge-session` record with an empty or missing `bridgeSessionId`, so dropping `&amp;&amp; record.BridgeSessionID != ""` would pass.
   - Evidence: `remotecontrol_test.go` (claudeengine) consts: `bridgeLine` always carries `"bridgeSessionId":"cse_01"`.
   - Fix: add a row `BridgeRecordWithoutASessionIDIsNoConnection`.
7. NIT, design.
   - Location: `internal/shuttleengine/remotecontrol.go:59,75`, the `sessionID == ""` guards in `remoteControlCheckpoint` and `awaitRemoteControl`.
   - Summary: no row runs with an empty session ID, which should give 0 or false and no read of the record.
   - Evidence: `TestRunner_AwaitRemoteControl` always passes `"session-1"` (`remotecontrol_test.go:92,96`).
   - Fix: add a row with an empty session ID, asserting zero reads.

Tool calls:
1. A grep of the orchcli tests for no-transcript or resume-error rows that combine with remote control, and of the claudeengine tests for the two new Claude methods. My context held the diff's test hunks but not the other rows of `start_test.go`, nor whether any claudeengine test calls those methods.
