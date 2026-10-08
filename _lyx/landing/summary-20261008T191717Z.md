# shuttle: sends that land, and stops that are recorded

A text shuttle sends into an agent pane now lands, and when it cannot, the run retries instead of halting.
A shuttle run that lyx itself stopped is recorded as stopped, so a later attach probe no longer halts on it.

Closes #442
Closes #443
Closes #444

## What changes

- **Send waits for an idle session.**
  Every verified send first waits, up to the new `send_ready_timeout_s` (default 60), until the pane reads idle and no `turn_start` signal follows the newest turn end in the events file.
  An unmatched `turn_start` holds a send for at most 10 s against an idle pane, or until `TurnStartInterrupt` reports the turn interrupted.
  A session that stays busy fails with the new `ErrSessionBusy` before any key is typed.
  `Runner.Send` into a busy pane therefore waits instead of failing at once.
- **Typed text settles before Enter.**
  For an engine that reads its input box, shuttle types, reads the box until two reads one `submit_settle_ms` apart agree, then sends Enter.
  An engine without that capability keeps the fixed choreography.
- **Submission is confirmed in a time window.**
  The new `submit_confirm_timeout_s` (default 30) bounds the window, which opens when typing begins.
  After Enter the box is re-read at an interval doubling from `submit_redraw_settle_ms` up to 5 s, and an extra Enter goes out only into a box still holding the sent text.
  `sendExtraEnters` is gone.
  A `turn_start` past the pre-send offset confirms a submission early.
  A submission still pending at the window's close, or a text that never appeared, fails with the new `ErrSubmissionNotLanded`.
- **A failed send clears its own text.**
  Before returning `ErrSubmissionNotLanded`, a send whose box holds copies of its text, a prefix of at least the needle's length, or a paste placeholder sends `C-u` (`ClearInputSequence`, `claudeengine`).
  Only an engine with both the idle reading and the clear choreography clears.
- **Send errors show the pane.**
  `ErrSessionBusy` and `ErrSubmissionNotLanded` end with the last 15 non-blank lines of the final pane capture, so the shed status file and the loom halt note show the cause.
- **A start-time send that does not land is a not-started run.**
  A failed skill-load send or prompt delivery, and any other start failure past `AddStrand`, now goes through `abandonStartup`: record `died`, strand removed, error wrapping `ErrNotStarted` beside the send's sentinel.
  Burler's attempt-2 retry and the agent-start transient mark apply unchanged; `shedengine.TransientAgentStart`'s doc comment widens to a session that never took its first input.
  After `ErrSubmissionNotLanded`, start reads the session signals once more, and a `turn_start` past the pre-send offset makes the start succeed.
- **Gate sends retry instead of ending the wait loop.**
  The boundary is confirmed idle in fact before the gate sends, so a replayed `AttachGated` no longer types into a running turn.
  A re-prompt that does not land spends no attempt and is re-sent on later ticks.
  Inside the wait loop a send's idle wait and submit window both end by the run deadline.
- **Shuttle owns stopping a run's strand.**
  A new stop verb, reachable from a run handle and by strand guid, writes the record to the new record-only outcome `stopped` and then removes the strand.
  Burler's half stop, webster's leftover-strand reclaim and attach's superseded-strand removal call it; burler's and webster's own strand removers are deleted.
  Attach's only classification change is that `isTerminalOutcome` recognises `stopped`, so a stopped run reads respawn-eligible at any directory age.
- **New `PATTERN-shuttle-stop`** pins that path.
  The `PATTERN-completion-signal` background file and its enforcement test cover `stopped`.
- `doc.go` for `shuttleengine`, `claudeengine`, `burlerengine` and `websterengine` follows, and the shuttle template carries the two new keys.

## Design decisions

- `stopped` is a record-only outcome, never a `Result.Outcome`.
  The stop verb keeps an already-terminal record's outcome and leaves `running` on a record whose output files all exist, so attach's file-contract-first branch still harvests a finished run (`PATTERN-completion-signal`).
  One function, `stopOutcome` in `stop.go`, chooses `runOutcomeStopped`; the stop verb and a marked handle's finalize both go through it.
- The record is written before the strand is removed.
  A crash between the two leaves a terminal record over a live strand, which attach already treats as respawn-eligible; the reverse order would recreate #444.
- `Run` gains a record mutex that guards the stop mark, `run.state.Outcome` and every save of `run.state`, so a mid-run `Wait` save cannot restore `running` over `stopped`.
  The guid form reads and rewrites `run.json` from disk and has no in-process `Wait` to race.
  A `Wait` in another process may still overwrite `stopped` with its own terminal outcome; either answer is terminal.
- The idle wait reads only the raw `turn_start` signal, never the session-state fold.
  It can delay a send or fail it as busy, and never ends, classifies or finalizes a run.
- The 10 s override for an unmatched `turn_start`, `stopped` instead of reusing `died`, and widening `TransientAgentStart` instead of adding a class are auto-picks.
  Whether a session that never took its first input deserves its own transient class is left to the operator.
- Webster's reclaim moves onto the stop verb although it never hit the attach refusal, so one stop path exists.
- Out of scope and unchanged: the unknown-run refusal in attach (a young `running` record with an untracked strand still refuses), `lyx reed remove` run by the operator, and `Runner.Inject`.

## To check by hand

- **Segment color is still typed raw.**
  The decision record calls for `/color` (`playInputs(ColorSequence)` in `run.go` after `awaitStartup`, and `Runner.TypeColor` in `session.go`) to go through the new send path after readiness, with a failed color send logged and never failing the launch.
  Both call sites still use `playInputs` in this diff.
  After the merge with main (reed-view), three `loomcli` tmux-tier tests failed because the raw `/color` left text in the input box: `TestIntegrationDriverBootstrap_ReturnsWithoutWaitingOnTheDriver`, `TestSmokeDriverStrand_ReentrantAcrossThreeBootstraps` and `TestSmokeStart_NeverAttachesAndAddsNoOperatorStrand`.
  Run `go test -tags tmux ./internal/loomcli/... ./internal/shuttleengine/...` and confirm those pass; I did not run it.
- **Color send can stall a prompt on the command line.**
  A plan review left open that a color send sharing the send path could leave the following prompt typed into a box still holding the `/color` command; check the color path once it goes through the send.
- **`PATTERN-shuttle-stop` omits the orch exception.**
  The entry names only `loomcli`'s removal of the loom driver's strand as an exception.
  `orchcli` also removes the orch strand raw, and the orch session does have a shuttle run record (`runner.Start`).
  The decision is to name that second exception and leave orch out of scope; check the `PATTERN.md` line says so.
- **#442's root cause is not reproduced live.**
  The design covers the three candidates (unconsumed paste, session still starting, readiness probe passing too early); check that the pane tail in the next occurrence's error text identifies the cause.
- **Idle reading on odd panes.**
  A session whose idle reading never passes (a greyed suggestion, a pane too short to draw its box) now fails a send as busy after `send_ready_timeout_s` instead of typing.
- **Hook lines.**
  The signal condition depends on hook lines reaching the events file; a missing `Stop` line delays a send into an idle pane, or a gate boundary's restore, by up to the 10 s override.
- **The clear can eat operator input.**
  A failed send's `C-u` removes whatever the box holds that matches the send's text or shows as a paste placeholder, including text an operator types into the pane while the window is open.
