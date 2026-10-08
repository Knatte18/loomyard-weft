# shuttleengine: sends that land, and stops that are recorded

A text sent into an agent pane now lands or fails with a cause, and a send that cannot land makes the run retry instead of halting.
A run that lyx itself stopped is now recorded as stopped, so a later attach probe no longer halts on it.

Closes #442
Closes #443
Closes #444

## What changes

Before this change a verified send could type into a session that was not ready, or into a running turn, and an Enter inside a typing burst was read as part of a paste and swallowed.
Burler stopped a half by removing its strand and left the record reading `running`; the next attach probe saw an untracked strand behind a young running record and refused, which is the duplicate-agent guard firing on a run lyx ended.

- **Idle wait before typing.**
  Every verified send waits until the session is idle in fact, polled with backoff up to the new key `send_ready_timeout_s` (default 60).
  Idle means the engine's idle reading passes, and, for an engine that reads session signals, no `turn_start` follows the newest turn end in the events file.
  Otherwise the send fails with the new sentinel `ErrSessionBusy` before any key is typed.
  `Runner.Send` into a busy pane now waits for idle like every other verified send, so `lyx shuttle send` into a busy agent takes up to the window before it fails.
- **Settle before Enter.**
  For an engine that reads its input box, the typed text must read the same twice, one `submit_settle_ms` apart, before Enter is sent.
  A collapsed paste placeholder counts as settled.
- **Confirm within a window.**
  After Enter the box is re-read at a doubling interval (from `submit_redraw_settle_ms`, up to 5 s) until the submission is confirmed or `submit_confirm_timeout_s` (new, default 30) closes.
  A further Enter goes out only into a box that still holds the sent text, as before.
  A `turn_start` past the pre-send offset also confirms a submission.
  `sendExtraEnters` is retired.
  A submission not confirmed, a text that never appeared in the pane, or a box that never settled fails with `ErrSubmissionNotLanded`.
- **Failures carry the pane.**
  Both sentinels end with the last 15 non-blank lines of the final pane capture, so the shed status file and the loom halt note show the cause.
- **Unlanded text is cleared.**
  Before returning `ErrSubmissionNotLanded` the send clears its own text out of the box through a new engine-owned choreography in `claudeengine`, so the next send does not fail busy against it.
  The clear runs only for an engine with both the idle reading and the clear choreography, only when the box holds copies of this send's text, a prefix of at least the needle's length, or a paste placeholder, and never uses a key that interrupts a turn.
- **A start-time send that does not land is a not-started run.**
  The skill-load send and the prompt delivery take the existing `abandonStartup` teardown on `ErrSessionBusy` or `ErrSubmissionNotLanded`: the record is finalized `died`, the strand removed, and the error wraps `ErrNotStarted` beside the send's sentinel.
  Every other start failure past `AddStrand` (the prompt-offset persist, for one) takes the same teardown, so no start error leaves a live strand behind a record reading `running`.
  Burler's attempt-2 retry and the agent-start transient mark at the shed boundary then handle it unchanged.
  The doc comment of `shedengine.TransientAgentStart` widens to a session that never took its first input; the class set is unchanged.
- **Gate sends retry.**
  In the wait loop the boundary is confirmed idle in fact before the gate sends anything, so `AttachGated`'s replayed events cannot cause a notice or re-prompt to be typed into a running turn.
  A re-prompt that fails busy or unlanded spends no attempt and is re-sent on a later tick.
  Inside the loop a send's idle wait and submit window both end by the run deadline.
- **A stop verb that records the stop.**
  `internal/shuttleengine` gains `Run.Stop` and `Runner.StopStrand` (by guid).
  The verb writes the record to the new record-only outcome `stopped` first, then removes the strand if reed reports it live.
  Burler's half stop, webster's leftover-strand reclaim and attach's superseded-strand removal call it, and their own strand removers are deleted.
  Attach classifies a `stopped` record as terminal, so the re-step that halted in #444 proceeds.
- **Config and docs.**
  The two new keys are in the shuttle template; zero or negative is a config error naming the key.
  `doc.go` of `shuttleengine`, `claudeengine`, `burlerengine`, `shedadapters` and `websterengine` is updated, and `PATTERN.md` gains `PATTERN-shuttle-stop`.

## Design decisions to review

- **`stopped` is written before the strand is removed.**
  A crash between the two leaves a terminal record over a live strand, which attach already reads as respawn-eligible; the reverse order would recreate the bug.
- **A record whose output files all exist is not marked.**
  The stop verb consults `allOutputFilesExist` first (`PATTERN-completion-signal`), so such a run keeps `running` and attach's file-contract-first branch still harvests it.
  `stopped` therefore means lyx ended the run before its output files existed.
  `runOutcomeStopped` has one return site, `stopOutcome`, and the enforcement test lists it as a negative verdict marker.
- **`Run` gains a record mutex.**
  The stop verb and the run's own `Wait` share `run.state` and `run.json`, so every save of `run.state` and the stop's mark take the lock, and a later mid-run save cannot restore `running`.
  A handle stopped through `Run.Stop` persists `stopped` in its `Wait` finalize unless the outcome is `done` or the output files exist.
  A `Wait` in another process may overwrite `stopped` with its own terminal outcome; either answer is terminal.
- **The hook signals get their first behaviour consumer.**
  The idle wait reads the raw `turn_start` and turn-end signals, never the fold's derived state, and can only delay a send or fail it as busy.
  An unmatched `turn_start` holds a send for at most 10 s when the pane reads idle on every poll, or until `SessionProber.TurnStartInterrupt` reports the turn interrupted; the override logs a Warn.
- **Reuse of `ErrNotStarted` and the transient class.**
  No new transient class and no new marking site (`PATTERN-transient-stop`).
  A prompt that lands late is caught twice before teardown: the box reading before the clear, and a final session-signal read.
- **`PATTERN-shuttle-stop` names one exception.**
  `internal/loomcli`'s removal of the loom driver's own strand stays a raw removal, since nothing reads the driver's record; it is out of scope here.
- **Out of scope.**
  The attach refusal for an untracked strand behind a young `running` record is unchanged, as are `lyx reed remove`, orch sessions and `Runner.Inject`.

## To check by hand

None of the following was run against a live agent.

- #442's root cause was not reproduced live.
  The change covers the three candidates (an unconsumed paste, a session still starting, a readiness probe passing too early), and the error text now names the cause on the next occurrence.
- A session whose idle reading never passes (a greyed suggestion, a pane too short to draw its box) now fails a send as busy after `send_ready_timeout_s` instead of typing.
- The idle condition depends on hook lines reaching the events file.
  A missing `UserPromptSubmit` line makes the signal condition vacuous; a missing `Stop` line delays a send, or the restoring of a gate boundary, by up to the 10 s override.
- The clear removes whatever the box holds that matches the send's text.
  An operator who types or pastes into the pane while the send's window is open loses that input when it starts like the send's text or shows as a paste placeholder.
- `stopLeftoverStrand` in `internal/websterengine/recoverbatch.go` and `strand.go`: a review judged the first version too broad, warning and stopping at sites that were not a leftover respawn, including `recoverSpawn` with an empty prior guid.
  A fix limits it to where a recorded strand is replaced; check that no remaining call site stops or warns for an empty guid, and that routine cleanup after a terminal digest does not log the respawn Warn.
- Widening `TransientAgentStart` rather than adding a class for "never took its first input" is a judgment call for the operator.
- `cmd/lyx/testdata/test-budget.yaml` changed to fit the new tests; confirm the budget moves are acceptable.
