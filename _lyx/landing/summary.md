# Shuttle sends land or retry; lyx-stopped runs are recorded stopped

A text sent into an agent pane now waits for an idle session, settles before Enter, and confirms the submission within a time window.
A send that cannot land fails with a typed error, and a start-time failure of that kind is a not-started run the shed retries instead of halting.
A run whose strand lyx itself stopped is recorded as `stopped`, so a later attach probe no longer refuses on it.

Closes #442
Closes #443
Closes #444

## Send (`internal/shuttleengine`, `claudeengine`)

- Every verified send first waits until the session is idle in fact: the pane shows the provider idle, and no `turn_start` hook signal follows the newest turn end in the run's events file.
  The wait is bounded by the new key `send_ready_timeout_s` (default 60) and fails with `ErrSessionBusy` before any key is typed.
  An unmatched `turn_start` (an interrupted turn, a missing `Stop` line) holds a send against an idle pane for at most 10 s, or less when the engine reports the turn interrupted.
- For an engine that reads its input box, the typed text must read the same on two consecutive reads, one `submit_settle_ms` apart, before Enter is sent.
- After Enter, the box is re-read with a doubling interval up to 5 s inside a window of `submit_confirm_timeout_s` (new key, default 30).
  A further Enter goes out only into a box still holding the sent text.
  `sendExtraEnters` is retired.
  A submission not confirmed in the window fails with `ErrSubmissionNotLanded`, after the send clears its own text out of the box.
- Both errors end with the last 15 non-blank lines of the final pane capture, so the halt note shows the cause.
- Start: a skill-load or prompt send that fails with either sentinel tears the run down through `abandonStartup` and wraps `ErrNotStarted`, which already maps to the agent-start transient class.
  Every other start failure past `AddStrand` takes the same teardown.
  The `shedengine.TransientAgentStart` doc comment is widened to a session that never took its first input; no class or marking site is added.
- Wait loop: the gate boundary is confirmed idle in fact before a send, a `turn_start` after it clears the boundary, and a re-prompt that fails with either sentinel is retried on later ticks without spending the gate's attempt budget.
  A send inside the loop shares the time left to the run deadline.
- The segment `/color` command that arrived with the merge of main now goes through the same send path after readiness, in `Start` and `Runner.TypeColor`.
  A failed color send is logged and never fails the launch.

## Stop (`stop.go`, burler, webster, attach)

- Shuttle gains one stop verb for a run's strand, by run handle and by strand guid.
  It writes the record-only outcome `stopped` first, then removes the strand, so a crash between the two leaves a terminal record over a live strand, which attach already treats as respawn-eligible.
- A record that is already terminal keeps its outcome, and a `running` record whose output files all exist keeps `running` so attach still harvests it as finished.
- `Run` gains a record mutex that every save of `run.state` takes, so a later mid-run save cannot restore `running` over `stopped`.
  Finalize takes its outcome from the one function `PATTERN-completion-signal` names, and `done` still wins when the output files exist.
- Burler's half stop, webster's leftover-strand reclaim and attach's superseded-strand removal call the verb; burler's and webster's own strand removers are gone.
- `isTerminalOutcome` recognising `stopped` is the only change to attach's classification.
  The unknown-run refusal in attach is unchanged.
- `PATTERN.md` gains `PATTERN-shuttle-stop`: no package outside `internal/shuttleengine` removes a shuttle run's strand except through the verb.
  It names two exceptions: loomcli's removal of the loom driver's strand and orchcli's removal of the orch strand, neither of which any attach probe or `Wait` reads.

## Design decisions to know

- Idle-in-fact reads only the raw `turn_start` and turn-end signals the hooks already write, never the session-state fold.
  It can delay a send or fail it as busy; it never ends or classifies a run.
- `ErrSubmissionNotLanded` also covers a text that never appeared in the pane, since its cause, a swallowed input, is the same miss.
- The clear after a failed submission runs only for an engine with both the idle reading and the clear choreography, only when a box reading just before shows this send's text, and never with a key that interrupts a running turn.
- `submit_settle_ms` and `submit_redraw_settle_ms` keep their keys, now the settle-read interval and the first backoff step; `send_ready_timeout_s` and `submit_confirm_timeout_s` reject 0 or negative.
- `doc.go` of `shuttleengine`, `claudeengine`, `burlerengine` and `websterengine` describes the new behavior.

## To check or run by hand

- The root cause of #442 was not reproduced live; the next occurrence should show its cause in the pane lines in the error text.
- `go test -tags tmux ./internal/loomcli/... ./internal/shuttleengine/...` must pass, in particular `TestIntegrationDriverBootstrap_ReturnsWithoutWaitingOnTheDriver`, `TestSmokeDriverStrand_ReentrantAcrossThreeBootstraps` and `TestSmokeStart_NeverAttachesAndAddsNoOperatorStrand`; the merged raw `/color` typing broke them before the fix.
- The segment color send in `internal/shuttleengine/run.go` (`Start`) and `session.go` (`Runner.TypeColor`) was carried over from a review finding: check that it goes through the send path after readiness and that a failed color send only logs.
- `PATTERN-shuttle-stop` in `PATTERN.md` was carried over from a review finding: check that it lists both the loom driver and the orch strand exceptions and that no other call site removes a shuttle run's strand.
- A session whose idle reading never passes (a greyed suggestion, a pane too short to draw its box) now fails a send as busy after `send_ready_timeout_s` instead of typing.
  `lyx shuttle send` into a busy agent waits up to that window before failing.
- A missing `UserPromptSubmit` hook line makes the signal condition vacuous, and a missing `Stop` line delays a send or a gate-boundary restore by up to 10 s.
- The clear removes whatever the box holds that starts like the send's text or shows as a collapsed paste placeholder, including an operator's input typed while the window is open.
- Whether a session that never took its first input deserves its own transient class instead of widening `TransientAgentStart` is left to the operator.
