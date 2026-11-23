---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: lens:code-affected-set
    location: internal/shuttleengine/claudeengine/remotecontrol.go#RemoteControlCheckpoint
    summary: "The new Claude reader spells the transcript file name as the literal `sessionID+\".jsonl\"` at two sites (lines 33 and 52), adding repeated literals of a provider path that PATTERN-code-conventions requires to be one named constant."
    fix: "In internal/shuttleengine/claudeengine/transcriptread.go declare one named constant for the transcript file extension (e.g. `transcriptExtension = \".jsonl\"`) or one helper `transcriptPathIn(projectDir, sessionID string) string` that joins it. Route both new sites in remotecontrol.go (RemoteControlCheckpoint line 33 and RemoteControlConnected line 52) through it. Leave the older sites unchanged."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/session.go#TurnOnRemoteControl
    summary: "When the transcript read fails after `/remote-control` was already pasted, TurnOnRemoteControl returns the error, so creel's typeInject default branch requeues the inject and types the command into the session again on every retry until the TTL runs out."
    fix: "In Runner.TurnOnRemoteControl, after the sequence was typed, handle an error from r.awaitRemoteControl by logging it at Warn with logger (strand guid, session id, err) and returning (false, nil), so the landed command is never reported as a failed delivery. State this in its doc comment. Add a row to TestRunner_TurnOnRemoteControl_ReportsTheRecordedConnection in internal/shuttleengine/remotecontrol_test.go with record.readErr set, asserting (false, nil) and exactly one paste."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/orchcli/start.go#relaunchStrand
    summary: "relaunchStrand now runs markLaunchedRemoteControl, a wait of up to 20 s, after the orch is launched but before registerOrchProcess and saveFreshLaunch, while it holds the start lock. A refused connection delays every start by the full wait, and an interrupted start in that window leaves a live orch that is neither recorded nor registered."
    fix: "In internal/orchcli/start.go relaunchStrand, move the block `if remoteControlName != \"\" && checkpointRead { c.markLaunchedRemoteControl(started, st.SessionID, checkpoint) }` from right after launchOrch to right after saveFreshLaunch returns nil, before the logger.Info launched line. The state is then saved and the process registered before the bounded wait. Keep the checkpoint read where it is, before launchOrch."
  - id: F4
    severity: LOW
    class: decision
    origin: lens:code-scope
    location: internal/orchcli/remotecontrol.go#turnOnRemoteControl
    summary: "The goal says the RC button shows remote control's real state, but a session whose remote control is later turned off or disconnects keeps its mark, and a click still answers that remote control is already on. The change description admits this narrowing, but no Added after Discussion entry records it."
    fix: "Leave the code in place and defer this finding to the run's parent. The parent either records with `lym loom decision add` that this task covers the connect and policy-refusal case only, or asks for disconnect handling as its own change. Recording the decision is outside the fixer's write surface."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/session.go#TurnOnRemoteControl
    summary: "When the paste lands but is unconfirmed (ErrNotLanded, which creel settles as unconfirmed rather than failed), TurnOnRemoteControl returns before waiting, so a session whose remote control does connect never gets its pane marked."
    fix: "In Runner.TurnOnRemoteControl, when typed is true and errors.Is(err, ErrNotLanded), still run r.awaitRemoteControl and return (connected, err). In internal/creelcli/delivery.go turnOnRemoteControl, call ops.MarkRemoteControlOn when connected is true even when err is non-nil, then return err. Add a row covering the unconfirmed paste."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/orchcli/remotecontrol_test.go#TestStart_LaunchPassesAndMarksRemoteControl
    summary: "No row covers a resume that falls back to a fresh launch with remote control on, so the `if started.SessionID != resumed { checkpoint = 0 }` reset in markLaunchedRemoteControl could be deleted and every test would still pass."
    fix: "Add a row to TestStart_LaunchPassesAndMarksRemoteControl with recordedSession \"sess-1\", the harness starter's resume failing with the no-transcript error the existing fallback test uses (start_test.go near line 1553), wantName \"neve::tst:orch\", wantAwait \"minted-id@0\" and wantMark true."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/claudeengine/remotecontrol_test.go#TestBridgeSessionRecordedIn
    summary: "No test reaches Claude.RemoteControlCheckpoint or Claude.RemoteControlConnected. Nothing pins the checkpoint as the transcript's size, a missing transcript as zero, or that both methods read the same session transcript path."
    fix: "Add a table test in internal/shuttleengine/claudeengine/remotecontrol_test.go that builds a fake Claude config dir under t.TempDir() the way the existing transcript-read tests do and writes a session transcript. Assert that RemoteControlCheckpoint returns its byte size and returns 0 for a missing transcript, and that RemoteControlConnected finds a bridge-session record appended past that checkpoint."
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/creelcli/delivery_test.go#recordingCommands
    summary: "No row covers the remote-control error paths. Nothing tests a checkpoint read that fails before the paste (nothing typed), creel returning a TurnOnRemoteControl error, or creel returning nil on a failed mark."
    fix: "Give recordingCommands an err field returned by TurnOnRemoteControl and a markErr field returned by MarkRemoteControlOn. In TestHubDelivery assert that runSessionCommand returns the TurnOnRemoteControl error and makes no mark, and that a mark failure on a connected session still returns nil. Give remoteControlRecord in internal/shuttleengine/remotecontrol_test.go a checkpointErr field, with a row asserting that TurnOnRemoteControl returns that error and pastes nothing."
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/remotecontrol_test.go#TestRunner_TurnOnRemoteControl_ReportsTheRecordedConnection
    summary: "The test claims to pin the pre-paste checkpoint, but the fake's checkpoint is a constant 42. A checkpoint taken after the paste would pass as well."
    fix: "Make remoteControlRecord.RemoteControlCheckpoint answer a value that changes once the paste lands, for example 42 while the fake reed records no PasteSubmitCalls and 99 after. Assert that every read was given 42."
  - id: F10
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md#Merge danger
    summary: "The blast radius names only reed. The diff also changes creel's delivery of the remote-control command, which now waits up to 20 s and decides the pane mark, and orch start and relaunch, which now wait up to 20 s for the connection."
    fix: "In _lym/landing/summary.md under Merge danger, widen Blast radius to name reed's RC mark, creel's remote-control command delivery and the orch start and relaunch path."
  - id: F11
    severity: NIT
    class: consistency
    origin: lens:code-tests
    location: internal/shuttleengine/remotecontrol_test.go#TestRunner_TurnOnRemoteControl_ReportsTheRecordedConnection
    summary: "The redundancy report lists this test as contained, but neither covering test asserts the connected value TurnOnRemoteControl returns or the checkpoint its read starts from. The test pins what coverage cannot see."
    fix: "Add `//testtiming:keep pins the connected result TurnOnRemoteControl returns and the checkpoint its read starts from` on the line directly above `func TestRunner_TurnOnRemoteControl_ReportsTheRecordedConnection`."
  - id: F12
    severity: NIT
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/claudeengine/remotecontrol_test.go#TestBridgeSessionRecordedIn
    summary: "No row has a bridge-session record with an empty or missing bridgeSessionId, so dropping that condition from isBridgeSessionRecord would pass."
    fix: "Add a row BridgeRecordWithoutASessionIDIsNoConnection whose transcript holds the line {\"type\":\"bridge-session\",\"sessionId\":\"s\"} plus a newline, want false."
  - id: F13
    severity: NIT
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/remotecontrol_test.go#TestRunner_AwaitRemoteControl
    summary: "No row passes an empty session ID, so the sessionID guards in remoteControlCheckpoint and awaitRemoteControl are unpinned."
    fix: "Add a sessionID field to the TestRunner_AwaitRemoteControl table, defaulting to \"session-1\", and a row with it empty that asserts checkpoint 0, connected false, zero record reads and no time waited."
  - id: F14
    severity: NIT
    class: consistency
    origin: handler
    location: internal/shuttleengine/remotecontrol_test.go:1
    summary: "The two new test files open with `package` and no file-level comment, while most sibling test files in both packages carry one."
    fix: "Add a one-line file-level comment, followed by a blank line, above `package shuttleengine` in internal/shuttleengine/remotecontrol_test.go (what it covers: the bounded remote-control wait and its checkpoint), and likewise above `package claudeengine` in internal/shuttleengine/claudeengine/remotecontrol_test.go (which transcripts read as a connected remote control)."
---

### [BLOCKING] Repeated transcript file-name literal in the new Claude reader

**Location:** internal/shuttleengine/claudeengine/remotecontrol.go#RemoteControlCheckpoint

**Issue:** Lines 33 and 52 each build the session transcript path as `filepath.Join(projectDir, sessionID+".jsonl")`.
PATTERN-code-conventions requires a name or path that can go stale to be one named constant, never a repeated literal.
Claude's transcript name is such a path, and the package already repeats it at `transcriptread.go:137`, `audit.go:45`, `background.go:48` and `resume.go:119,196`.
The diff adds to that debt.
The constraints section makes every violation of a listed entry BLOCKING.

### [MEDIUM] A read failure after the paste makes creel type /remote-control again

**Location:** internal/shuttleengine/session.go#TurnOnRemoteControl

**Issue:** Once the sequence is typed, `return r.awaitRemoteControl(...)` passes any transcript read error up through `creelcli.turnOnRemoteControl` (`if err != nil { return err }`) to `hubDelivery.Command`.
In `internal/creelengine/deliver.go` `typeInject`, an error that is neither nil, ErrNotLanded nor ErrBusy reaches the default branch, which calls `store.requeue`.
The worker then types `/remote-control` into the live session again on each retry, although the command already landed.
The creelcli doc comment says a mark failure is not fatal because the command has landed, and the same reasoning applies to a read failure.

### [MEDIUM] The orch start blocks on the connection wait before it saves and registers the launch

**Location:** internal/orchcli/start.go#relaunchStrand

**Issue:** `markLaunchedRemoteControl` now calls `awaitRemoteControl`, which polls for up to `remoteControlConnectWait` (20 s).
It runs right after `launchOrch` and before `registerOrchProcess` and `saveFreshLaunch`, and the start path holds the start lock (start.go:523) throughout.
In the policy-refused case this task targets, every `lym beam start` with remote control on waits the full 20 s.
For those 20 s the new orch runs without its strand and session saved in the orch state and without its process registered.
An interrupt there leaves an unrecorded live orch, where the window used to be milliseconds.

### [LOW] The disconnect case of "real state" is narrowed without a recorded decision

**Location:** internal/orchcli/remotecontrol.go#turnOnRemoteControl

**Issue:** The goal's first line asks that the RC button show remote control's real state.
The diff lights the button only on a recorded connection, which fixes the reported policy case.
A later disconnect, or remote control turned off inside Claude Code, still leaves `@lym_rc` set and keeps the "already on" refusal.
The change description's Design decisions section names this as unchanged, but the decision record carries no Added after Discussion entry narrowing the goal.

### [LOW] An unconfirmed paste skips the connection wait

**Location:** internal/shuttleengine/session.go#TurnOnRemoteControl

**Issue:** `if err != nil || !typed { return false, err }` returns before waiting when `sendSessionSequence` reports ErrNotLanded.
Creel settles that outcome as unconfirmed rather than failed (deliver.go `case errors.Is(err, ErrNotLanded)`), since the command may well have landed.
If it did and remote control connects, the pane stays dim. The failure is safe, but the button then shows the wrong state.

### [LOW] No row covers the checkpoint reset on a resume that fell back to a fresh launch

**Location:** internal/orchcli/remotecontrol_test.go#TestStart_LaunchPassesAndMarksRemoteControl

**Issue:** `markLaunchedRemoteControl` resets the checkpoint to 0 when `started.SessionID != resumed`.
No row resumes a session that fails with the no-transcript error while remote control is on, so removing the reset would read the fresh session's transcript from the old session's offset and still pass.

### [LOW] Claude's checkpoint and connection methods are untested

**Location:** internal/shuttleengine/claudeengine/remotecontrol_test.go#TestBridgeSessionRecordedIn

**Issue:** The reach report lists `Claude.RemoteControlCheckpoint` and `Claude.RemoteControlConnected` as reached by no mapped test, and no claudeengine test calls them.
Untested behaviors: the checkpoint as the file size, zero for a missing transcript, and both methods resolving the same transcript path.

### [LOW] The remote-control error paths have no rows

**Location:** internal/creelcli/delivery_test.go#recordingCommands

**Issue:** `recordingCommands` always returns `(r.connected, nil)` and a nil mark error, and `remoteControlRecord.RemoteControlCheckpoint` never fails.
Three behaviors are unpinned: creel returning a TurnOnRemoteControl error, creel succeeding despite a failed mark, and TurnOnRemoteControl typing nothing when the pre-paste checkpoint read fails.

### [LOW] The pre-paste ordering of the checkpoint is not pinned

**Location:** internal/shuttleengine/remotecontrol_test.go#TestRunner_TurnOnRemoteControl_ReportsTheRecordedConnection

**Issue:** The doc comment says the test pins the point the record reached before the paste.
The fake answers a constant 42 regardless of when it is asked, so a checkpoint taken after the paste passes too.

### [LOW] The merge-danger blast radius names less than the diff changes

**Location:** _lym/landing/summary.md#Merge danger

**Issue:** The blast radius names only `reed`.
Beyond reed, the diff changes creel's delivery of the remote-control command: it now waits for the connection, decides the pane mark, and on a read error requeues the command.
It also adds a bounded wait to the orch start and relaunch path.

### [NIT] Mark the contained TurnOnRemoteControl test as kept

**Location:** internal/shuttleengine/remotecontrol_test.go#TestRunner_TurnOnRemoteControl_ReportsTheRecordedConnection

**Issue:** The redundancy report lists this test as contained by `TestRunner_TurnOnRemoteControl_PastesTheNamedCommandThroughTheGuardedSend` and `TestRunner_AwaitRemoteControl`.
The first discards the returned bool (`session_test.go:954`), and the second calls `AwaitRemoteControl` directly.
So this test alone pins the connected result and the checkpoint hand-off, and it needs a `//testtiming:keep` marker.

### [NIT] No row has a bridge-session record without a bridgeSessionId

**Location:** internal/shuttleengine/claudeengine/remotecontrol_test.go#TestBridgeSessionRecordedIn

**Issue:** Every bridge row carries `"bridgeSessionId":"cse_01"`, so the `record.BridgeSessionID != ""` condition in `isBridgeSessionRecord` is unpinned.

### [NIT] No row has an empty session ID in the await test

**Location:** internal/shuttleengine/remotecontrol_test.go#TestRunner_AwaitRemoteControl

**Issue:** Every row passes `"session-1"`, so the `sessionID == ""` guards in `remoteControlCheckpoint` and `awaitRemoteControl` are unpinned.

### [NIT] The new test files lack a file-level comment

**Location:** internal/shuttleengine/remotecontrol_test.go:1

**Issue:** `internal/shuttleengine/remotecontrol_test.go` and `internal/shuttleengine/claudeengine/remotecontrol_test.go` open directly with `package`.
Most sibling test files in both packages open with a file-level comment saying what they cover.

## Rejected

- code-affected-set finding 2 (the 20 s `remoteControlConnectWait` breaks PATTERN-daemon-wakeups' keep-warm cap): rejected as a constraint violation. The constant times out one provider-record read inside a command, well under a minute. It is not the cap on an agent's wait, which the keep-warm rule governs so that a wait cannot outlast the prompt cache. The real cost, the start blocking under its lock before saving, is kept as F3.
- code-correctness side note on a rewritten, shorter transcript being read whole: the doc comment of `bridgeSessionRecordedIn` states the behavior, and no path in the diff rewrites a live session's transcript shorter. Not filed.
