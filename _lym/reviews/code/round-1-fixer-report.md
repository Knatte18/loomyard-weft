---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
---

# Round 1 fixer-report

F4 is a `decision`-class finding held by the parent, so it is left out.
The first gate run was refused before any test ran, for two comments that wrapped mid-sentence or reached 120 characters.
A follow-up commit breaks the `turnOnRemoteControl` doc comment in `internal/creelcli/delivery.go` and the doc comment of `TestClaude_RemoteControlReadsTheSessionTranscript` into one sentence per line.
No code changed in it.
A later gate run listed reach notes only, and I added no test for them.
`runSessionCommand`, `turnOnRemoteControl` and `MarkRemoteControlOn` in `internal/creelcli` are already driven by the "Each command reaches its session method" subtest of `TestHubDelivery`, which this round extended.
The notes name them as unmapped only because the reach map of `internal/creelcli` was built before the round's commits, and `wireCLI` is code this round did not change.

## What changed

- **F1** (commit `claudeengine: fix F1`): `transcriptPathIn(projectDir, sessionID)` in `transcriptread.go` joins the transcript file name once.
  `RemoteControlCheckpoint` and `RemoteControlConnected` in `claudeengine/remotecontrol.go` both use it; the older sites are unchanged, as the fix said.
- **F3** (commit `orchcli: fix F3`): `markLaunchedRemoteControl` now runs right after `saveFreshLaunch` returns nil in `relaunchOrch`, before the `launched` log line.
  The checkpoint read stays before `launchOrch`.
- **F6** (commit `orchcli: fix F6`): a row in `TestStart_LaunchPassesAndMarksRemoteControl` resumes `sess-1` with the no-transcript error and expects `minted-id@0` and a mark.
  The test gained a `resumeErr` field, and it reads the last started spec since the fallback starts two sessions.
- **F2, F5, F7, F8, F9, F11, F12, F13, F14** (one commit, since they share test files):
  - F2: `Runner.TurnOnRemoteControl` logs a failed read of the record after the paste at Warn and returns `(false, nil)`.
  - F5: a paste that fails with `ErrSubmissionNotLanded` still waits for the connection and returns `(connected, err)`.
    The reviewer's `ErrNotLanded` is creel's name for that outcome; shuttle's sentinel is `ErrSubmissionNotLanded`.
    The other creel-side not-landed causes, `ErrSentTextInDraft` and `ErrForeignBoxText`, leave the command unsubmitted, so they do not wait.
    `creelcli.turnOnRemoteControl` marks whenever `connected` is true and then returns the error.
    The doc comments of both state this.
  - F7: `TestClaude_RemoteControlReadsTheSessionTranscript` covers the checkpoint size, zero for a missing transcript, and a bridge-session record past the checkpoint.
    It is not parallel because it points `HOME` at a fake config directory.
  - F8: `recordingCommands` has `turnOnErr` and `markErr`, and `remoteControlRecord` has `checkpointErr`.
    Rows cover a turn-on error with no mark, a failed mark that still returns nil, and a failed checkpoint read that pastes nothing.
  - F9: the fake record answers 42 until a paste lands and 99 after, and the test asserts every read was given 42.
  - F11: the `//testtiming:keep` marker sits on the line above the test.
  - F12: row `BridgeRecordWithoutASessionIDIsNoConnection`.
  - F13: an `emptySessionID` field on the await table, with a row asserting checkpoint 0, no reads and no wait.
    I used a bool rather than a defaulting string, so the empty value stays distinguishable from the default.
  - F14: file-level comments on both new test files.
- **F10**: the Blast radius line in `_lym/landing/summary.md` names reed's RC mark, creel's remote-control command delivery, and the orch start and relaunch path.
  The file is under `_lym`, so it is edited and not committed.

## Self-check

Every fork returned a report.

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal, LOW, F5 waits the full 20 s when the command provably never ran: disputed, `every fix worse`.
- fix-proposal, note on F9's closure reading `reed.PasteSubmitCalls` without the reed's lock: not a finding, since the paste runs on the caller's goroutine and the gate passed.

## Disputed

- fix-proposal's LOW on F5 (self-check finding, not a review finding), ground `every fix worse`.
  The 20 s wait after `ErrSubmissionNotLanded` is bounded, and the result is correct (no mark, error returned).
  Skipping the wait would restore the review's F5 premise: a session whose paste landed unconfirmed and whose remote control connected would stay unmarked.
  Evidence: `internal/shuttleengine/session.go` (`TurnOnRemoteControl`), `internal/shuttleengine/send.go:556`.

## Deferred

- F4 is not mine: it is `decision`-class and goes to the parent, who either records with `lym loom decision add` that this task covers the connect and policy-refusal case only, or asks for disconnect handling as its own change.

## Rulings

None reached me this round.
