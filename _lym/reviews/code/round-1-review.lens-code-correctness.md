```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. I ran each function and branch the diff adds or changes with a concrete case.

**Per function or branch: the case I ran, and the result**
- `creelcli.sessionCommandOps.MarkRemoteControlOn` with guid "g": forwards to `reed.MarkRemoteControlOn`. Correct.
- `creelcli.turnOnRemoteControl`:
  - `TurnOnRemoteControl` errors (busy): the error is returned and creel requeues. Correct.
  - Returns (false, nil): logs at Info and returns nil. Correct.
  - Returns (true, nil), then the mark errors: logs at Warn and returns nil. Correct.
  - The command was typed, then the transcript read errors: the error is returned, creel requeues and types the command again. Wrong (finding 1).
- `shuttleengine.Runner.TurnOnRemoteControl`:
  - Name "a\nb": refused before anything is typed. Correct.
  - `toldErr` is set: refused. Correct.
  - Empty `state.SessionID`: the checkpoint is 0 and the wait returns false without waiting. Correct.
  - Engine without `RemoteController`: `typed` stays false and it returns (false, nil). Correct.
  - Engine with no reader: types the command and returns false. Correct.
  - The send returns `ErrNotLanded` after the paste: returns (false, err) without waiting, so a connection that does land stays dim (finding 3).
  - The send succeeds, then the read fails: the error is returned after the command landed (finding 1).
- `Runner.RemoteControlCheckpoint` and `Runner.AwaitRemoteControl`:
  - `toldErr` is set: returns the error. Correct.
  - Unknown engine name: the lookup error is returned. Correct.
- `Runner.remoteControlCheckpoint` with a non-reader engine, or `sessionID` "": returns (0, nil). Correct.
- `Runner.awaitRemoteControl`:
  - Never connects, fake clock: 41 reads, then false at 20 s. Correct.
  - Connects on the first read: returns at once. Correct.
  - Read error: wrapped and returned. Correct.
- `Claude.RemoteControlCheckpoint`:
  - Missing file: returns 0. Correct.
  - `sessionProjectDir` errors when HOME is unset: the error is returned. Correct.
- `Claude.RemoteControlConnected` with a missing transcript: returns false. Correct.
- `claudeengine.bridgeSessionRecordedIn`:
  - Checkpoint 0 on an empty file: `ReadBytes` gives an empty line and EOF, so false. Correct.
  - Checkpoint equal to the file size: false. Correct.
  - Checkpoint larger than the file: the whole file is read. Matches the doc, though a rewritten transcript can relight the button on an old record (documented, not filed).
  - Seek lands mid-line: that fragment fails to unmarshal and is skipped. Correct.
  - Complete record with no trailing newline: counted. Correct.
- `claudeengine.isBridgeSessionRecord`:
  - Invalid JSON: false.
  - Right type with an empty `bridgeSessionId`: false.
  - The type string quoted inside a message: false.
  - All correct.
- `orchcli.launchRemoteControlCheckpoint`:
  - Empty name or empty session: returns (0, true). Correct.
  - The read errors: returns (0, false), so the launch skips the mark. Correct.
- `orchcli.markLaunchedRemoteControl`:
  - `markRemoteControl` is nil: returns. Correct.
  - Resume fell back to a fresh launch (`started.SessionID` != resumed): checkpoint is reset to 0. Correct.
  - Wait errors, or reports no connection: logged, no mark. Correct.
  - Connects: marked. Correct.
  - The wait runs before the state is saved and the process registered (finding 2).
- `orchcli.relaunchStrand`:
  - `checkpointRead` false: skips the mark. Correct.
  - The launch errors: returns before the wait. Correct.
- `orchcli.turnOnRemoteControl` and `remoteControlOnCmd`:
  - Pane already marked: refused. Correct.
  - The inject fails: error. Correct.
  - Success: returns the fields and marks nothing. Correct.

**Findings**

1. **MEDIUM, design.** Location: `internal/shuttleengine/session.go` `TurnOnRemoteControl`, last line (`return r.awaitRemoteControl(...)`), and `internal/creelcli/delivery.go` `turnOnRemoteControl` (`if err != nil { return err }`).
   - Summary: a failed transcript read after `/remote-control` has already been pasted is returned as a delivery failure. Creel's `typeInject` default branch then calls `store.requeue` and types the command into the session again, on every retry until the inject's TTL runs out.
   - Evidence: diff `session.go` lines 1103–1106; `internal/creelengine/deliver.go` lines 418–437 (the default branch requeues); diff `delivery.go` lines 57–60.
   - A failure after the command landed should be logged and leave the pane dim, not returned.

2. **MEDIUM, design.** Location: `internal/orchcli/start.go` `relaunchStrand`, lines 791–806.
   - Summary: `markLaunchedRemoteControl` now blocks for up to 20 s (`remoteControlConnectWait`) while holding the start lock. It does so after the new orch is launched but before `registerOrchProcess` and `saveFreshLaunch`.
   - Effect: an interrupted or killed `lym beam start` in that window, or any refused connection, leaves a live orch that is neither recorded nor registered. The window used to be milliseconds.
   - Evidence: `start.go` lines 786–806 and diff `remotecontrol.go` lines 196–204.
   - The wait belongs after the state is saved and the process registered.

3. **LOW, design.** Location: `internal/shuttleengine/session.go` `TurnOnRemoteControl`, `if err != nil || !typed { return false, err }`.
   - Summary: when the paste lands but is unconfirmed (`ErrNotLanded`, which creel settles as unconfirmed), no wait runs. A session whose remote control does connect therefore never gets its pane marked.
   - Evidence: diff `session.go` line 1103; `deliver.go` lines 427–431.

Tool calls:
- `grep -A40` of `sendSessionSequence` in `internal/shuttleengine/session.go`: to see whether the `typed` closure runs before the send can fail, and which errors can follow a paste. My context had only the diff hunk.
