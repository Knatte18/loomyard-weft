# Light the RC button only once Remote Control connects

The RC button lit green as soon as lym asked for Remote Control, whether or not the session got it.
Where an organization's policy disables Remote Control, the button showed on and a click answered that it was already on.

Now the pane's `@lym_rc` mark is set only once the session's own transcript records the connection.
Claude Code writes a `bridge-session` record naming the remote session when it connects, and writes none for a refused request.
A refused request leaves the button dim, so a later click is not told Remote Control is already on.

Closes #189

## What changes

- `shuttleengine` gains an optional `RemoteControlReader` engine capability, implemented for Claude in `claudeengine/remotecontrol.go`.
  `RemoteControlCheckpoint` returns the transcript's size before the request.
  `RemoteControlConnected` scans the transcript from that offset for a `bridge-session` record with a non-empty `bridgeSessionId`.
  A transcript shorter than the checkpoint is read whole, and a line still being written is not read as a record.
- `Runner.AwaitRemoteControl` polls that record for up to 20 seconds, every 500 ms, on the runner's clock.
  `Runner.TurnOnRemoteControl` now takes the checkpoint, pastes the command, waits, and returns `(connected bool, err error)`.
  A paste not seen to land still waits for the connection, and reports it beside `ErrSubmissionNotLanded`.
  A failed read after the paste is logged and reports not connected, since the command has landed.
- `creelcli` marks the pane only when `TurnOnRemoteControl` reports connected.
  A failed mark is logged and does not fail the command.
- `orchcli`'s `rc-on` click no longer marks the pane itself: it queues the command through creel, which marks once the connection is recorded.
  A launch of the orch with remote control takes a checkpoint before it resumes the session, and marks after its state is saved and its process registered.
  A fresh launch instead of a resume counts the whole record.
- `reedengine` drops the engine-less `MarkPaneRemoteControlOn`, so every mark goes through `Engine.MarkRemoteControlOn`.
  Its comments now say the mark means the connection was recorded.

## Design decisions

- The check reads what the provider did, never what lym asked for.
  An engine without `RemoteControlReader` never reports a connection, so its button stays dim.
- Scope is the button's state at connect and on a policy refusal.
  A later disconnect or turn-off of Remote Control is not handled and belongs to a separate change.
- The 20-second bound sits on the paths that wait: the creel command delivery and the orch launch.
  A connection lands about a second after the request, so the bound is only paid in full on a refusal.

## To check by hand

- Nothing was run against a real organization policy.
  In a hub whose policy disables Remote Control, launch the orch and click RC: the button should stay dim and reed should not claim it is on.
  Where Remote Control is allowed, the button should light within a few seconds.
- The `bridge-session` record name and its `bridgeSessionId` field are taken from Claude Code's transcript format and are not covered by any contract.
  Check them against the Claude Code build in use.
- The orch launch waits for the connection (up to 20 seconds) inside `relaunchStrand`, after the save and register but before the launch reports.
  Check that this delay is acceptable on `lym orch start`.
- A read failure after a paste in `TurnOnRemoteControl` returns not connected with no error.
  Check that a command whose paste landed is never queued a second time because of it.
- The transcript filename suffix `.jsonl` is built in `transcriptPathIn` in `claudeengine/transcriptread.go`.
  Check that no other place in `claudeengine` still repeats the literal.

## Merge danger

**Door:** two-way
**Blast radius:** orch
