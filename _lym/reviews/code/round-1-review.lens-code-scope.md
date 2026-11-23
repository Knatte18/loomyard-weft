```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. It traces each change in the diff to a goal point of the board entry and checks that every goal point is built.

**Decisions.** These are the goal points in the board entry under `## Goal`. The record has no `Added after Discussion` entries.
- **G1.** The RC button shows Remote Control's real state for the session.
- **G2.** Where the organization's policy disables Remote Control, the button shows it off or unavailable.
- **G3.** A click does not claim Remote Control is on.

**Per change:**
- **C1. Claude reads the `bridge-session` record.** Covers `claudeengine/remotecontrol.go` and its `doc.go` lines. Traces to G1 and G2: it is the signal that the connection really happened.
- **C2. Shuttle gains the reader and a bounded wait.** Covers the `RemoteControlReader` interface in `shuttleengine/engine.go`, `shuttleengine/remotecontrol.go`, `TurnOnRemoteControl` returning `connected`, and the `shuttleengine/doc.go` lines. Traces to G1 and G2.
- **C3. Creel marks the pane only once a connection is recorded.** Covers `creelcli/delivery.go`: `sessionCommandOps`, `turnOnRemoteControl` and the interface change. Traces to G1 and G2, and to G3 on the click path.
- **C4. An orch launch marks the pane only on a recorded connection past the pre-launch checkpoint.** Covers `orchcli/remotecontrol.go` `launchRemoteControlCheckpoint` and `markLaunchedRemoteControl`, `start.go:785-793`, and the `cli.go` seams. Traces to G1 and G2; this is the reported orch-pane case.
- **C5. The `rc-on` verb no longer marks the pane.** Covers `orchcli/remotecontrol.go` `turnOnRemoteControl` and the `remoteControlOnCmd` help text. Traces to G3: the click no longer lights the button, and a later click no longer answers "already on".
- **C6. Forced by C5:** removal of `reedengine.MarkPaneRemoteControlOn` in `reedengine/remotecontrol.go`, whose only caller was the click verb.
- **C7. Forced by C3 to C5:** the comment changes in `reedengine/bar.go` and `reedengine/remotecontrol.go`, and the file header of `orchcli/remotecontrol.go`.
- **C8. Forced by C1 to C5:** the test changes in `creelcli/delivery_test.go`, `orchcli/remotecontrol_test.go`, `shuttleengine/remotecontrol_test.go`, `shuttleengine/session_test.go`, `shuttleengine/enginedispatch_test.go` and `claudeengine/remotecontrol_test.go`.

No change in the diff is unrequested scope.

**Per decision:**
- **G1:** built by C1 to C4, partly; see the finding below.
- **G2:** built by C1 to C4.
- **G3:** built by C5, with C3.

**Findings:**

1. **Severity LOW, class `scope`.** Location: `internal/orchcli/remotecontrol.go:145`, the `turnOnRemoteControl` "already on" refusal, together with the `@lym_rc` mark lifecycle in `internal/reedengine/remotecontrol.go`.
   - **Summary:** G1 is built only for the moment remote control connects. A session whose remote control is later turned off inside Claude Code, or disconnects, keeps the RC button lit, and a click still answers "remote control is already on". So the button does not show the session's real state in that case. The summary's "Design decisions" admits this, but no `Added after Discussion` entry in the decision record covers the narrowing.
   - **Evidence:**
     - `_lym/landing/summary.md:1255-1257` says "Turning remote control off inside Claude Code still leaves the mark set, as before … No transcript record of a disconnect was found to read."
     - `internal/orchcli/remotecontrol.go:145-146` keeps the "already on" refusal.
     - The decision record's `## Goal` ("The RC button shows Remote Control's real state for the session") has no added entry narrowing it.
   - **Fix:** either clear the mark on a recorded disconnect, or have the run's parent record this narrowing of G1 with `lym loom decision add`.

Tool calls: none; my inherited context held everything this lens needed.
