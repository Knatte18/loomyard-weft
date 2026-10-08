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
  A session that stays busy fails with the new `ErrSessionBusy`, before any key is typed.
  `Runner.Send` into a busy pane therefore waits instead of failing at once.
- **Typed text settles before Enter.**
  For an engine that reads its input box, shuttle types, reads the box until two reads one `submit_settle_ms` apart agree, then sends Enter.
  An engine without that capability keeps the fixed choreography.
- **Submission is confirmed in a time window.**
  The new `submit_confirm_timeout_s` (default 30) bounds the window.
  After Enter the box is re-read at a doubling interval from `submit_redraw_settle_ms` up to 5 s, and an Enter goes out only into a box still holding the sent text.
  `sendExtraEnters` is gone.
  A `turn_start` past the pre-send offset confirms a submission early.
  A submission not confirmed in the window fails with the new `ErrSubmissionNotLanded`, and so does a sent text that never appears in the pane.
  Before returning it, the send clears its own text from the box, so the next send does not fail as busy against it.
- **Failures show the cause.**
  Both sentinels end with the last 15 non-blank lines of the final pane capture, so the shed status file and the loom halt note show what the pane held.
- **A start-time send that does not land is a not-started run.**
  The skill-load send and prompt delivery route `ErrSessionBusy` and `ErrSubmissionNotLanded` through `abandonStartup`: the record is `died`, the strand removed, and the error wraps `ErrNotStarted` beside the send's sentinel.
  Burler's attempt-2 retry and the agent-start transient mark at the shed boundary then apply.
  Every other start failure past `AddStrand` takes the same teardown, so none leaves a live strand behind a record reading `running`.
  If a `turn_start` appears by the signal read before teardown, the start succeeds instead.
- **Mid-run gate sends retry.**
  The wait loop confirms the writer idle before evaluating the gate, so a replayed attach no longer types into a running turn.
  A re-prompt that fails busy or not-landed spends no attempt and is re-sent on a later tick, bounded by the run deadline.
  A send inside the loop takes one budget, the time left until the deadline.
- **Shuttle owns stopping a run's strand.**
  A new stop verb, reachable from a run handle and by guid, writes the record-only outcome `stopped` before it removes the strand.
  Burler's half stop (through the handle), webster's leftover-strand reclaim and attach's superseded-strand removal use it, and burler's and webster's own strand removers are deleted.
  Attach reads `stopped` as terminal, so the re-step that halted in #444 proceeds.
  A record already terminal keeps its outcome, and a `running` record whose output files all exist keeps `running` so attach still harvests it as finished.
- **Docs.**
  `PATTERN-shuttle-stop` is new, and `doc.go` of `shuttleengine`, `claudeengine` and `burlerengine` describe the changes.
  `shedengine.TransientAgentStart` widens to "an agent session that never became ready or never took its first input".

## Design decisions to review

- **`stopped` is record-only.**
  It is never a `Result.Outcome`.
  The stop verb consults `allOutputFilesExist` first, through one function that is the single return site of `runOutcomeStopped`, as `PATTERN-completion-signal` requires; `completionsignal_enforcement_test.go` is updated to match.
- **Record before removal.**
  A crash between the two leaves a terminal record over a live strand, which attach already treats as respawn-eligible.
- **Record lock on `Run`.**
  The stop verb and the run's own `Wait` share `run.state` and `run.json`, so every save of `run.state` takes the new record mutex, and a handle stopped through the verb is marked so its `Wait` finalize persists `stopped`.
  A `Wait` in another process may still overwrite `stopped` with its own terminal outcome.
- **First behaviour consumer of the hook signals.**
  The idle condition reads only the raw `turn_start` and turn-end signals, never the session-state fold.
  It can only delay a send or fail it as busy.
- **Transient class on an auto-pick.**
  Widening `TransientAgentStart` rather than adding a class was not confirmed by the operator.
- **Out of scope.**
  `Runner.Inject`, orch sessions, the unknown-run refusal in attach for untracked or binding-cleared runs, `lyx reed remove`, and loomcli's raw removal of the loom driver's strand, which `PATTERN-shuttle-stop` names as its one exception.

## To check by hand

- **Segment color is not yet sent through the new path.**
  `playInputs(…, ColorSequence(…))` in `internal/shuttleengine/run.go` and `Runner.TypeColor` in `internal/shuttleengine/session.go` still type `/color` raw; they arrived with the merge of `main`.
  A raw `/color` left in the input box made three tmux-tier tests fail against the stricter readiness: `TestIntegrationDriverBootstrap_ReturnsWithoutWaitingOnTheDriver`, `TestSmokeDriverStrand_ReentrantAcrossThreeBootstraps` and `TestSmokeStart_NeverAttachesAndAddsNoOperatorStrand`.
  The intended fix sends the color after readiness through the send path, with a failed color send logged and never failing the launch.
  Run `go test -tags tmux ./internal/loomcli/... ./internal/shuttleengine/...` and confirm these three pass; the tmux tier was not run for this description.
- **Root cause of #442 is not reproduced live.**
  The design covers the three named candidates (unconsumed paste, session still starting, readiness probe passing too early), and the pane lines in the error make the next occurrence diagnosable.
- **A session whose idle reading never passes** (a greyed suggestion, a pane too short to draw its box) now fails a send as busy after the window instead of typing.
- **The clear removes input.**
  An operator who types or pastes into the pane while a send's window is open loses that input if it starts like the send's text or shows as a collapsed paste placeholder.
- **Missing hook lines.**
  A missing `UserPromptSubmit` line makes the signal condition vacuous, and a missing `Stop` line delays a send, or a gate boundary's restore, by up to 10 s.
- **Config.**
  Confirm the template comments for `send_ready_timeout_s`, `submit_confirm_timeout_s`, `submit_settle_ms` and `submit_redraw_settle_ms`, and that 0 or a negative value for either new key is a config error naming that key.
