---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/reedengine/panebin.go:283
    summary: The rc-on and rc-bar verb names are declared as constants in orchcli but repeated as string literals in reedengine's composed lines, against PATTERN-code-conventions' one-named-constant rule
    fix: Declare the two verb words once, as exported constants in internal/reedengine/bar.go beside RemoteControlLit and RemoteControlUnlit (for example RemoteControlOnVerb = "rc-on" and RemoteControlBarVerb = "rc-bar"). In internal/reedengine/panebin.go use them in composeRemoteControlClickCommand's words slice (line 283) and in composeRemoteControlStateCommand's returned line (line 304) in place of the literals. In internal/orchcli/remotecontrol.go set remoteControlOnVerb and remoteControlBarVerb from those reedengine constants, or delete them and reference the reedengine constants directly at every use, so each verb name is declared exactly once.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lym/landing/summary.md:50
    summary: The change description claims the click always sends the command lit or not, that a Codex orch still draws a dim RC button, and that the button shows whenever the worktree records an orch strand; the diff does none of these
    fix: In _lym/landing/summary.md, replace the Design-decisions bullet "The click always sends the command, lit or not" with the behavior the code has, that a click refuses and sends nothing while the record reads on for the live strand, its way forward naming /remote-control typed by hand in the orch pane. Delete the Open-risks bullet saying a Codex orch still draws a dim RC button, and state instead that an orch on an engine with no remote control writes no record, so no RC button is drawn. In the RC-button bullet, say the button shows only while the worktree records an orch strand and the remote-control record names that strand. Mention that a start adopting an already-running strand the record does not name records it off.
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/orchcli/start_test.go:764
    summary: The already-running branch of startOrch records an unrecorded live strand off, but no test asserts it, since the RC word is checked only for rows without a live watcher
    fix: >-
      In TestStart_LiveStrand (internal/orchcli/start_test.go), move the remoteControlBarWordAt assertion out of the `if !c.watcher || c.releases` block so it runs for every row that reaches the success path, including "with a watcher does nothing" and "a stopping watcher that never exits is reported live", which then expect reedengine.RemoteControlUnlit. Also add one row with watcher true and seedRecord {Strand: "g1", On: true} expecting wantLit, so the already-running branch's keep-the-record path is pinned as the spawn-watcher branch's is.
---

The range is `git diff d385d53e..HEAD` over the seven commits from `5a0c8639` to `2b07ea6e`.
The touched packages pass `lym gate test` at tier0 (4395 passed, 0 failed).

### [BLOCKING] RC verb names repeated as literals across packages

**Location:** `internal/reedengine/panebin.go:283`

**Issue:** `internal/orchcli/remotecontrol.go` declares `remoteControlBarVerb = "rc-bar"` and `remoteControlOnVerb = "rc-on"` and uses them as the cobra `Use` names.
`internal/reedengine/panebin.go` composes the lines that call those verbs with the literals `"orch", "rc-on"` (line 283) and `" orch rc-bar --dir "` (line 304).
PATTERN-code-conventions says a name that can go stale is "one named constant, declared once and referenced everywhere, never a repeated literal".
If either verb is renamed in orchcli, the pinned click and state lines silently call a verb that no longer exists, and the bar shows no button.
The Slack lines' `" slack bar"` literal is earlier debt of the same kind and does not license a new instance.
`orchcli` already imports `reedengine`, so the constants can live in reedengine beside `RemoteControlLit`/`RemoteControlUnlit` and be referenced from both packages.

### [MEDIUM] The change description contradicts the diff on the click, Codex and the button's visibility

**Location:** `_lym/landing/summary.md:50`

**Issue:** The summary says "The click always sends the command, lit or not", which it justifies as recovery for a session turned off inside Claude Code.
`turnOnRemoteControl` (`internal/orchcli/remotecontrol.go:145`) refuses and sends nothing while `RemoteControlOn` reads on, as the parent's added decision requires, and `TestTurnOnRemoteControl`'s `AlreadyOn` row pins that.
Its Open-risks bullet "On a Codex orch the RC button still draws, dim" is also false.
`recordRemoteControl` writes nothing for an engine without `RemoteController`, so `remoteControlBarWordAt` prints no word and the bar draws no button, as the `EngineWithoutRemoteControl` row of `TestStart_LaunchPassesAndRecordsRemoteControl` pins.
"It shows only while the session's worktree records an orch strand" leaves out the condition that the record must name that strand.
The description does not mention that a start adopting an already-running strand records it off.

### [MEDIUM] Already-running adoption's record write is untested

**Location:** `internal/orchcli/start_test.go:764`

**Issue:** `startOrch`'s default branch (`internal/orchcli/start.go:698`) calls `recordUnrecordedRemoteControl` after adopting a live strand with a live watcher.
That is the common case when an orch that predates this change is brought up again, and it is what makes the RC button appear there.
`TestStart_LiveStrand` checks the RC word only inside `if !c.watcher || c.releases`, so the two rows that reach `actionAlreadyRunning` never assert it.
Deleting that call fails no test.
The spawn-watcher branch's call is pinned by the two new rows; the already-running branch is not.

## Focus verdicts

- `Runner.TurnOnRemoteControl`: `TestRunner_TurnOnRemoteControl_PastesTheNamedCommandThroughTheGuardedSend` pins the pasted `/remote-control <name>` through `checkPastedSessionCommand`'s guarded-send rows, the refusal of `\n` and `\r\n` names with nothing typed, the no-op returning nil on `readyCyclerEngine` (no `RemoteController`), and the told-path error returned before reed is touched.
  No defect.
- `turnOnRemoteControl`: a click on a lit record refuses with nothing queued, and the record is left on (`AlreadyOn` row).
  Two orch strands refuse with their guids and `lym reed remove`.
  A failed record write keeps the success and adds a `warning` field.
  All three match the parent's decision. No defect.
- Adopt and Codex: the spawn-watcher adopt keeps a record that already names the strand lit, and records a strand it does not name as off.
  A Codex orch writes no record on launch or adopt, so `rc-bar` prints nothing and no button is drawn.
  The only gap is the already-running branch's test coverage, which is F3.
